---
layout:     post
title:      Windows和Linux下改变JDK版本方法
category: blog
description: 最近在配置几个工具，经常需要切换JDK的版本，在这里做总结
---

最近在配置几个JAVA工具，但是工具所使用的JDK版本并不相同，导致工具不能成功运行。这时候就需要我们频繁切换JDK版本，下面简单总结一下Linux和Windows下切换JDK的方法。

## Linux切换JDK

Linux下的JDK切换比较简单：

* 首先，在你的Linux系统中已经装有多个版本的JDK

* 然后，执行下述指令

> sudo update-alternatives --config java

* 执行这条指令之后会弹出一个已装JDK版本的列表，输入你要选择JDK版本的数字即可

* 最后，可以用 java -version 进行确认是否切换JAVA版本，同样的方法也适用于javac,javadoc的切换

## Windows切换JDK

Windows下的JAVA切换比较麻烦，需要手动修改JAVA_HOME路径

* 首先，在找到设置环境变量的位置：我的电脑 -- 右键属性 -- 更改设置 -- 高级 -- 环境变量

* 然后，找到环境变量JAVA_HOME，把路径换成新的JDK路径

* 之后，环境变量Path中的JAVA/bin路径也要进行修改，本来以为这样可以切换成功，然而JAVA版本还是原来的

* 最后，将JAVA/bin的路径调整到C:\ProgramData\Oracle\Java\javapath前面，切换成功