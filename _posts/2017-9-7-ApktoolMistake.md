---
layout:     post
title:      解决Linux中使用ApkTool遇到问题
category: blog
description: 最近由于工作需要，用到了安卓逆向工具ApkTool,在这里记录遇到的问题
---

## 遇到问题

在Linux中使用IntelliDroid工具时，按要求配置好环境之后，始终无法成功运行该工具内部的ApkTool，导致后续的安卓静态分析不正确。其中错误信息如下：

```
I: Baksmaling...
I: Loading resource table...
I: Loaded.
W: Could not decode attr value, using undecoded value instead: ns=android, name=
versionCode, value=0x0000000a
W: Could not decode attr value, using undecoded value instead: ns=android, name=
versionName, value=0x00000011
Exception in thread "main" java.lang.NullPointerException
        at java.io.Writer.write(Unknown Source)
        at brut.androlib.res.util.ExtMXSerializer.writeAttributeValue(ExtMXSeria
lizer.java:38)
        at org.xmlpull.mxp1_serializer.MXSerializer.attribute(MXSerializer.java:
673)
        at org.xmlpull.v1.wrapper.classic.XmlSerializerDelegate.attribute(XmlSer
ializerDelegate.java:106)
        at org.xmlpull.v1.wrapper.classic.StaticXmlSerializerWrapper.writeStartT
ag(StaticXmlSerializerWrapper.java:267)
        at org.xmlpull.v1.wrapper.classic.StaticXmlSerializerWrapper.event(Stati
cXmlSerializerWrapper.java:211)
        at brut.androlib.res.decoder.XmlPullStreamDecoder.decode(XmlPullStreamDe
coder.java:46)
        at brut.androlib.res.decoder.ResStreamDecoderContainer.decode(ResStreamD
ecoderContainer.java:34)
        at brut.androlib.res.decoder.ResFileDecoder.decode(ResFileDecoder.java:1
00)
        at brut.androlib.res.AndrolibResources.decode(AndrolibResources.java:114
)
        at brut.androlib.Androlib.decodeResourcesFull(Androlib.java:93)
        at brut.androlib.ApkDecoder.decode(ApkDecoder.java:98)
        at brut.apktool.Main.cmdDecode(Main.java:120)
        at brut.apktool.Main.main(Main.java:57)
```

在查找过资料之后，原因主要是因为ApkTool的版本问题，很多解决方案将ApkTool更新到最新版本之后就可以成功运行，因此，我们也尝试了这种方法。

我们将IntelliDroid中最初提供的Apktool及Apktool.jar文件替换为官方上下载的最新版，重新运行该工具的启动脚本，运行成功。

## ApkTool安装

按照官网上的教程进行操作，将*apktool*和*apktool.jar*放入*/usr/local/bin*中，即可使用Apktool工具的指令。
