---
layout:     post
title:      Adb安装及配置
category: blog
description: 最近由于配置安卓分析工具，用到了adb工具，在这里简单记录
---

## adb简介

adb全名Andorid Debug Bridge. 顾名思义, 这是一个Debug工具.然而, 为何称之为Bridge呢? 因为adb是一个标准的CS结构的工具, 是要连接开发电脑和你的调试手机的.包含如下几个部分:

* Client端, 运行在开发机器中, 即你的开发PC机. 用来发送adb命令.
* Deamon守护进程, 运行在调试设备中, 即的调试手机或模拟器.
* Server端, 作为一个后台进程运行在开发机器中, 即你的开发PC机. 来管理PC中的Client端和手机的Deamon之间的通信.

## adb在Linux下的安装

在Linux下安装adb工具还是比较方便的，只需要使用apt-get命令就可以，命令如下：

```
sudo apt-get install android-tools-adb
```

如果原来的源中没有这个应用，则先更新再安装：

```
sudo add-apt-repository ppa:nilarimogard/webupd8
sudo apt-get update
sudo apt-get install android-tools-adb
```

输入adb shell若进入Linux shell环境即安装成功。

## adb在Windows下的安装

在Windows中安装adb工具就相对复杂一些了，因为还需要我们进行环境变量的配置才可以。

* 首先，我们要先下载android-sdk-tools安装包，安装后得到Android SDK Manager，这里就有我们想要的adb.exe。（这里我遇到一个问题，最新的android-sdk-tools安装之后好像只能通过命令行的方式进行Android API，tools的下载，我没搞懂怎么用，就在网上找到了一个2015年的旧版，安装后可用）

* 安装好SDK之后，进行环境变量的配置，创建系统变量adb并设为adb的所在目录，并在系统变量Path中添加%adb%，下面是我配置的例子

> adb 设为 F:\Android\android-sdk\platform-tools

> path + %adb%

* 最后，在cmd中输入adb查看是否配置成功

## adb使用时遇到问题

我在配置工具时，该工具使用了adb命令进行apk的安装和文件的上传。这时经常会遇到一个问题：*Read-only file system*，最开始我搞错了debug的方向，后来发现其实非常简单。

* 首先进入Linux shell

> adb shell

* 在shell中执行命令

> mount -o remount -o rw /

这样会将device中的所有文件都改为可读写属性

* 现在再执行 adb push 命令就可以成功上传到想要的位置了
