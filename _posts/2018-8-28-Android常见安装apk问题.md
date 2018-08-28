---
layout:     post
title:      Android常见安装apk问题
category: blog
description: 使用soot插桩app之后遇到了很多apk安装的问题，总结
---

## Android常见安装apk问题

#### 1. [**INSTALL_FAILED_DEXOPT**]

这个问题一般都是apk本身出现了问题，可能是插桩的代码有问题或者api版本等问题，检查一下代码或插桩的字节码等，可以解决这个问题。

#### 2.[**INSTALL_FAILED_**UID_CHANGED]

这个问题一般是在安装一个apk时，这个应用在之前卸载的过程中有残留，我们需要在adb shell中，把残留的数据文件手动删除解决这个问题。

> $ cd /data/data
>
> $ rm -rf <package name>

#### 3.[INSTALL_FAILED_OLDER_SDK]

安装设备的sdk版本与应用的最低要求apk不匹配，这种情况就只能换一个更高版本的系统设备了= =

#### 4.[INSTALL_PARSE_FAILED_NO_CERTIFICATES] 

这种是没有对插桩后的apk进行签名导致的问题，我们需要首先在自己电脑上生成keystore文件，然后用jarsigner进行签名，下面是一段java调用命令行进行应用签名的代码；

```java
// @param apk: 签名apk路径 
public void signApk(String apk) {
    try {
        // jarsigner is part of the Java SDK
        System.out.println("Signing " + apk);
        String cmd = "jarsigner -verbose -digestalg SHA1 -sigalg MD5withRSA -storepass android -keystore "+System.getProperty("user.home")+"/.android/debug.keystore "
            + apk + " androiddebugkey";
        System.out.println("Calling "+ cmd);
        Process p = Runtime.getRuntime().exec(cmd);
        printProcessOutput(p);
    } catch (IOException e) {
        System.out.println(e.getMessage() + e);
    }
}
```

#### 5.[INSTALL_FAILED_TEST_ONLY]

这个是在Android Studio更新的比较新的版本之后，点击run运行之后在build文件夹中的apk文件不是可以直接安装运行的apk文件，用这个文件安装之后就会出现这个问题；需要专门build才可以生成不是test_only的apk文件。

<u>暂时先想到这里，有些可能遗漏了，之后想到了再补充...</u>

---

