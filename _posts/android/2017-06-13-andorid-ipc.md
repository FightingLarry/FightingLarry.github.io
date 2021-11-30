---
layout: post
category: Android
title:  "Android IPC 机制"
tags: [Android,设备唯一标识码]
---

# Inter-Process Communication

IPC是Inter-Process Communication的缩写，含义就是进程间通信或者跨进程通信。



# Android序列化机制——Serializable接口

Serializable是Java提供的一个序列化接口，它是一个空接口，为对象标准的序列化和反序列化操作。使用Serializable来实现序列化相当简单，一句话即可。

```java
public class User implements Serializable {
  private static final long seriaVersionUID = 673000231720105497L
}
```



# Binder



# Bundle



# 文件共享



# AIDL



# Messenger



# ContentProvider



# Socket



