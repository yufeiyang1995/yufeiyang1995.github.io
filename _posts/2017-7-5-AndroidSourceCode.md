---
layout:     post
title:      Android系统源码的下载和编译
category: blog
description: 由于要开始进行Android Runtime方面的研究工作，所以先把源码弄下来看看。
---

## 源码下载

### 环境配置

我使用的是最新的系统Ubuntu 16.04，其中硬盘内存最好设置的大一些，因为源码及其编译之后的内容都会占用比较大的空间。同时，内存最好也设大一些，避免发生编译时内存不足的情况。

下面介绍需要安装的配置：

1. 首先，下载源码一定要安装git，可安如下步骤进行操作：

```
sudo apt-get install git 
git config –global user.email "test@test.com" 
git config –global user.name "test"
```

其中test@test.com和test分别是你的邮箱和用户名

2. 之后，我们需要下载Repo工具用于后面源码的下载

```
mkdir ~/bin
PATH=~/bin:$PATH
curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
chmod a+x ~/bin/repo
```

这里不对Repo的工作原理进行介绍，想要了解可以在网上查找一些相关资料。

3. 然后，为了编译Android源代码我们还需要进行java的安装，因为Android 5.x-6.0采用的都是openjdk 7，所以我们也需要安装openjdk 7。如果我们直接apt-get，会发现无法下载，我们需要先设置ppa：

```
sudo add-apt-repository ppa:openjdk-r/ppa 
sudo apt-get update
```

然后执行安装命令

```
sudo apt-get install openjdk-7-jdk
```

在安装好Java之后，如果你的系统中有其他的Java版本你可能需要进行Java版本的切换，命令如下：

```
sudo update-alternatives --config java
sudo update-alternatives --config javac
```

### 源码下载

1. 建立源码文件夹

```
mkdir source
cd source
```

2. 初始化仓库

```
repo init -u https://aosp.tuna.tsinghua.edu.cn/platform/manifest
```

或附带版本号

```
repo init -u https://aosp.tuna.tsinghua.edu.cn/platform/manifest -b android-4.0.1_r1
```

或先下载初始包，推荐使用这种方法，可以减少网络故障造成的问题

```
wget -c -t 0 https://mirrors.tuna.tsinghua.edu.cn/aosp-monthly/aosp-latest.tar 
tar -vxzf aosp-latest.tar
cd aosp
```

3. 同步源代码到本地

```
repo sync
```

可能会遇到几次失败，历经多次执行这条指令之后终于成功。

### 编译源代码

1. 进入源码目录初始化编译环境

```
source build/envsetup.sh
```

2. 选择目标

```
lunch
```

执行lunch指令之后，会出现所有的编译目标选项，我们可以进行选择，例如：

```
lunch aosp_arm64-eng
```

3. 开始编译

通过make指令进行代码编译

```
make
```

也可以通过-j参数，设置多线程编译

```
make -j8
```

运行顺利的话，几个小时之后可以看到 make completed successfully (01:18:45(hh:mm:ss)) ，表示编译成功

4. 运行模拟器

在编译完成之后,就可以通过以下命令运行Android虚拟机了,命令如下：

```
source build/envsetup.sh
lunch(选择刚才你设置的目标版本,比如这里了我选择的是2)
emulator
```

不出意外,在等待一会之后,你会看到运行界面

### 遇到问题总结


