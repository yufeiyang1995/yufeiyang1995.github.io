---
layout:     post
title:      FlowDroid介绍，源码分析及功能修改
category: blog
description: 做东西用到了FlowDroid，但是需要进行一些改动，对这些进行记录
---

## FlowDroid介绍，源码分析及功能修改

#### FlowDroid介绍

FlowDroid是目前对Android app进行污点分析效果最好的工具之一。污点分析的目的其实很简单，就是为了检查是否应用中是否存在从污点源到泄漏点的数据流。但是它的优点在于它构建的数据流精度很高，可以对上下文，流，对象和字段敏感，从而使得分析结果非常精确。

它实现精准分析的原因有几点：1. 它对Android声明周期进行了比较完整的构建，例如Activity中的OnCreate，OnResume等。通过抽象一个dummyMain作为分析的入口来支持Android应用的分析； 2. 它实现了精准的数据流分析，其中包含前向污点分析和后向别名分析。他们的实现其实都是基于heros的数据流分析框架来实现的，后面会简单介绍。这里面的算法比较复杂，我的理解是这两种分析都是满足上下文敏感和流敏感的，后向分析的算法提供了对象敏感和字段敏感的支持，这个算法比较复杂，我理解的也并不是很深； 3. 它支持简单的native code的污点分析

不足之处有：1. 不能对组件间（Intent）的污点传播进行分析 2. 隐式流问题 3. native code不能完美支持

#### FlowDroid安装

主页：[mainpage](https://blogs.uni-paderborn.de/sse/tools/flowdroid/)

下载：[soot-infoflow-android](https://github.com/secure-software-engineering/soot-infoflow-android)

其中安装时需要配置5个项目：`heros`， `jasmin`， `soot`， `soot-infoflow`和`soot-infoflow-android`，在github上都可以很容易的找到项目，`git clone`到本地之后，我们依次把他们导入ide当中（我使用的是eclipse），然后运行`soot-infoflow-android`中路径`src/soot/jimple/infoflow/android/TestApps`中的`Test.java`，必须配置输入参数为android jar包和要分析的apk文件，运行后没有报错即可看到分析结果。

其中我在配置时，也遇到了一些报错的问题，但是没有截图记录所以只能简单的总结一下：1. 缺少必要的jar包，导致项目无法编译，从网上下载指定的jar导入报错项目当中；2. 项目版本不对应，其实这个问题还是挺常见的，因为这几个开源项目都是分别开发的，所以有些版本不对应，我的方法是都调整为1.5对应的版本后成功。

>  现在FlowDroid的项目进行了迁移：[FlowDroid](https://github.com/secure-software-engineering/FlowDroid)，在Readme当中，只需要简单的maven运行之后就可以使用，但是我在尝试时maven一直在报错，所以就还是用1.5的版本进行后面修改。

#### FlowDroid源码分析及修改

首先，介绍一下我们的修改目的，扩展FlowDroid对普通的class文件进行分析的支持，不由dummyMain作为分析的入口点，而是从一个方法开始进行污点分析，其中方法参数可能时污点源，我们对这一可能进行了支持。

然后，我们先介绍一下FlowDroid的大概实现原理，在上面介绍的那个Test.java中，可以找到程序的main函数，我们从这里开始介绍。在对要分析的apk文件进行解析之后，runAnalysis方法开始进行分析。

1. application配置

```java
final SetupApplication app;
if (null == ipcManager){
	app = new SetupApplication(androidJar, fileName);
}
else{
    app = new SetupApplication(androidJar, fileName, ipcManager);
}
			
// Set configuration object
app.setConfig(config);
```

2. 先配置一下src&sink，在calculateSourcesSinksEntrypoints中会进行一些关于Android生命周期的分析，从而计算出分析的入口点，然后runInfoFlow，运行分析

```java
// 设置source和sink
app.setTaintWrapper(taintWrapper);
app.calculateSourcesSinksEntrypoints("SourcesAndSinks.txt");
// 开始分析
final InfoflowResults res = app.runInfoflow(new MyResultsAvailableHandler());
```

3. 之后在runInfoflow中，开始计算数据流

```java
info.computeInfoflow(apkFileLocation, path, entryPointCreator, sourceSinkManager);  // 计算数据流
```

4. 这里已经进入soot-infoflow项目中分析，里面会首先初始化分析的环境`initializeSoot`，并配置好soot的一系列信息，然后`runAnalysis`，在这个函数里就实现了前向和后向的数据流分析，并输出分析的结果。下面介绍几个关键步骤：

```java
...
// 构建ICFG,dataflow分析的基础
iCfg = icfgFactory.buildBiDirICFG(config.getCallgraphAlgorithm(), config.getEnableExceptionTracking());  
...
// Initialize the data flow problem
InfoflowProblem forwardProblem = new InfoflowProblem(manager, aliasingStrategy, aliasing, zeroValue);
// We need to create the right data flow solver
IInfoflowSolver forwardSolver = createForwardSolver(executor, forwardProblem);
...
// 检查是否有source和sink
for (SootMethod sm : getMethodsForSeeds(iCfg))
    sinkCount += scanMethodForSourcesSinks(sourcesSinks, forwardProblem, sm);
...
// 求解问题
forwardSolver.solve();
```

#### 我的修改

为了完成上面的实现需求，主要修改的有两方面：一个是入口点的设置，可以支持对于某一个method进行分析，另一个就是source源可以是method的某一参数。

我们先增减程序的参数：分析的classes路径，android jar包，要分析的method名称，以及该方法的污点参数，然后对最开始的main函数进行修改，记录参数的信息，然后在`soot.jimple.infoflow.AbstractInfoflow`的initializeSoot方法中，利用soot创建jimple语句的方法，构建了java的main方法并在方法里调用需要分析的method，这样soot就可以以这个方法作为入口分析，如果不这样进行调用，该分析方法在soot环境中不是activitybody，因此不能进行分析。

```java
Options.v().set_main_class(className);
SootClass entryClass = Scene.v().getSootClass(className);
Type stringArrayType = ArrayType.v(RefType.v("java.lang.String"), 1);
Body body;
SootMethod mainMethod = entryClass.getMethodByNameUnsafe("main");
if (mainMethod != null) {
    entryClass.removeMethod(mainMethod);
    mainMethod = null;
}
// Create the method
mainMethod = Scene.v().makeSootMethod("main", Collections.singletonList(stringArrayType), VoidType.v());

// Create the body
body = Jimple.v().newBody();
body.setMethod(mainMethod);
mainMethod.setActiveBody(body);

// Add the method to the class
entryClass.addMethod(mainMethod);
entryClass.setApplicationClass();
mainMethod.setModifiers(Modifier.PUBLIC | Modifier.STATIC);

// Add a parameter reference to the body
LocalGenerator lg = new LocalGenerator(body);
Local paramLocal = lg.generateLocal(stringArrayType);
body.getUnits()
    .addFirst(Jimple.v().newIdentityStmt(paramLocal, Jimple.v().newParameterRef(stringArrayType, 0)));
Local newLocal = lg.generateLocal(Scene.v().getType(className));
body.getUnits().addLast(Jimple.v().newAssignStmt(newLocal,
                                                 Jimple.v().newNewExpr(Scene.v().getRefType(className))));
SootMethod toInit = entryClass.getMethod("void <init>()");
body.getUnits().addLast(Jimple.v().newInvokeStmt(
    Jimple.v().newSpecialInvokeExpr(newLocal,toInit.makeRef())));


SootMethod toInvoke = null;
boolean mSign = false;
for(int i = 0;i < entryClass.getMethods().size();i++) {
    System.out.println("class: "+entryClass.getMethods().get(i));
    if(entryClass.getMethods().get(i).getName().equals(InputArgsConfig.getMethodName())) {
        mSign = true;
        break;
    }
}
if(mSign)
    toInvoke = entryClass.getMethodByName(InputArgsConfig.getMethodName());        
else
    return;

// TODO 多参数
int argsNum = InputArgsConfig.getArgsNum();
List<Value> argList = new ArrayList<>();
System.out.println("argNum: " + argsNum);
for(int i = 0;i < argsNum;i++) {
    Local arg = lg.generateLocal(Scene.v().getType("java.lang.Object"));
    argList.add(arg);
}
Local arg = lg.generateLocal(Scene.v().getType("java.lang.Object"));    

body.getUnits().addLast(Jimple.v().newInvokeStmt(
    Jimple.v().newVirtualInvokeExpr(newLocal,
                                    toInvoke.makeRef(),
                                    //Collections.singletonList(arg))
                                    argList)
));
body.getUnits().addLast(Jimple.v().newReturnVoidStmt());
```

然后，在`soot.jimple.infoflow.android.source.AccessPathBasedSourceSinkManager`中修改方法<getSourceInfo>，里面识别了source源分析的入口，所以我们添加污点参数作为源的支持。当有污点参数时，我们获取该参数的初始化语句，并将语句的左边符号设置为污点变量向下传播即可完成这一功能。

```java
if(sCallSite instanceof DefinitionStmt) {
    ArrayList<String> taintArgs = InputArgsConfig.getTaintArgsList();
    if(taintArgs != null) {
        for(int i = 0;i < taintArgs.size();i++) {
            if(sCallSite.toString().contains("parameter"+taintArgs.get(i)))
                return new SourceInfo(AccessPathFactory.v().createAccessPath(
                    ((DefinitionStmt) sCallSite).getLeftOp(), true));
        }
    }
}
```

关键的修改就是这些，其他地方就是一些不复杂的小修改，修改量并不是很大，但是需要掌握soot的原理，jimple语句的创建等才能进行实现，同时因为FlowDroid的实现是多线程的，所以在理解程序功能时的调试有一定难度。