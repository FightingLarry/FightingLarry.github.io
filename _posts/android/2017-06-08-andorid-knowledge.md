---
layout: post
category: Android
title:  "Android知识点整理"
tags: [Android]
---



## Android源码知识

### [GeniusVJR/LearningNotes Android知识点](https://github.com/GeniusVJR/LearningNotes)



### [LittleFriendsGroup/AndroidSdkSourceAnalysis](https://github.com/LittleFriendsGroup/AndroidSdkSourceAnalysis)

![img](https://github.com/yuxingxin/AndroidWidgetClassGraph/raw/master/img/android.jpg)

[ViewGroup 源码解析](https://github.com/LittleFriendsGroup/AndroidSdkSourceAnalysis/blob/master/article/ViewGroup%20%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90.md)

[Service源码解析](https://github.com/asLody/SourceAnalysis/blob/master/Service%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90.md)

[Binder源码解析](https://github.com/xdtianyu/SourceAnalysis/blob/master/Binder%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90.md)

[Handler源码解析](https://github.com/maoruibin/HandlerAnalysis)

[Bundle源码解析](https://github.com/ASPOOK/BundleAnalysis)

[LocalBroadcastManager源码解析](https://github.com/czhzero/AndroidSdkSourceAnalysis/blob/master/article/LocalBroadcastManager%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90.md)

[LruCache源码解析](https://github.com/LittleFriendsGroup/AndroidSdkSourceAnalysis/blob/master/article/LruCache%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90.md)

[MediaPlayer源码解析](https://github.com/lber19535/SourceAnalysis/blob/master/Media%20Player%20%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90.md)

[SharePreferences源码解析](http://blog.csdn.net/yanbober/article/details/47866369)

### [yipianfengye/androidSource](https://github.com/yipianfengye/androidSource)

[android源码解析之（八）-->Zygote进程启动流程](http://blog.csdn.net/qq_23547831/article/details/51104873)

> 大家都知道android系统的Zygote进程是所有的android进程的父进程，包括SystemServer和各种应用进程都是通过Zygote进程fork出来的。Zygote（孵化）进程相当于是android系统的根进程，后面所有的进程都是通过这个进程fork出来的，而Zygote进程则是通过linux系统的init进程启动的，也就是说，android系统中各种进程的启动方式init进程 –>...

### [xdtianyu/android-msm-hammerhead-3.4-marshmallow源码](https://github.com/xdtianyu/android-msm-hammerhead-3.4-marshmallow)



## Android项目整理

[yipianfengye/androidProject](https://github.com/yipianfengye/androidProject)

[Android产品研发（七）-->Apk热修复](http://blog.csdn.net/qq_23547831/article/details/51587927)

> 去年一整年android社区中刮过了一阵热修复的风，各大厂商，逼格大牛纷纷开源了热修复框架，恩，产品过程中怎么可能没有bug呢？重新打包上线？成本太高用户体验也不好，咋办？上热修复呗。好吧，既然要开始上热修复的功能，那么就得调研一下热修复的原理。下面我将分别讲述一下热修复的原理，各大热修复框架的比较，以及自身产品中热修复功能的实践



## Android 资源大全中文版

[ jobbole/awesome-android-cn ](https://github.com/jobbole/awesome-android-cn)

Android资源整理



## 记录android开发过程中遇到的问题。

[ayyb1988/android-issues](https://github.com/ayyb1988/android-issues)

#### 282 setCompoundDrawablesWithIntrinsicBounds()

网上一大堆setCompoundDrawables()方法无效不显示的问题，然后解决方法是setBounds，需要计算大小…不用这么麻烦，用setCompoundDrawablesWithIntrinsicBounds()这个方法最简单！

####282 更新媒体库文件

以前做ROM的时候经常碰到一些第三方软件（某音乐APP）下载了新文件或删除文件之后，但是媒体库并没有更新，因为这个是需要第三方软件主动触发。

[![img](https://camo.githubusercontent.com/c9109b1067da6e743f4a5bb79079b2cc6ff65ff3/687474703a2f2f6d6d62697a2e717069632e636e2f6d6d62697a2f65344a696243677a587636517567596665314c68675134526556567047597a6b68737a6433595551534d626e78666962744e69614a51453736696248764a7565696371416e656d5251747a385a49336f6867474d3037314a516c772f3634303f77785f666d743d706e672674703d7765627026777866726f6d3d352677785f6c617a793d31)](https://camo.githubusercontent.com/c9109b1067da6e743f4a5bb79079b2cc6ff65ff3/687474703a2f2f6d6d62697a2e717069632e636e2f6d6d62697a2f65344a696243677a587636517567596665314c68675134526556567047597a6b68737a6433595551534d626e78666962744e69614a51453736696248764a7565696371416e656d5251747a385a49336f6867474d3037314a516c772f3634303f77785f666d743d706e672674703d7765627026777866726f6d3d352677785f6c617a793d31)

## Android MVVM

[awesome-android-mvvm](https://github.com/chiclaim/awesome-android-mvvm)