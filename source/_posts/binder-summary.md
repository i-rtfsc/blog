---
title: Binder概述
categories:
  - Android
type:
  - Binder
  - IPC
description: Binder基本概念、原理，Binder机制系列文章列表
comments: true
abbrlink: d6aea0f1
date: 2021-10-07 17:26:26
---
<!--more-->
<meta name="referrer" content="no-referrer"/>

# 简介
在Android的中，我们知道每个应用都是一个独立的进程，有一个独立的虚拟机，应用和应用之间的内存是不能共享数据的，但是我们用到Activity、Service、Context、provider等功能恰恰做了很多数据传输。这就涉及到了我们了解的进程间通信机制-IPC。
在Android中常常使用的进程间通信有共享内存、管道、信号处理、socket、Binder等，大部分场景下使用的IPC都是Binder。Binder相较于传统IPC来说更适合于Android系统，具体原因的包括如下三点：

- Binder本身是C/S架构的，这一点更符合Android系统的架构
- 性能上更有优势：管道，消息队列，Socket的通讯都需要两次数据拷贝，而Binder只需要一次。
- 安全性更好：传统IPC形式，无法得到对方的身份标识（UID/GID)，而在使用Binder IPC时，这些身份标示是跟随调用过程而自动传递的。Server端很容易就可以知道Client端的身份，非常便于做安全检查。

​

# 架构
![avatar](https://cdn.nlark.com/yuque/0/2020/png/1759879/1601210476189-c4fc849a-1c9e-44f6-bdb9-248960ed5209.png)
从上图可以看出，Binder的实现分为：

- 驱动层
- framework层
   - C++
   - JNI
   - Java

驱动层位于Linux内核中，它提供了最底层的数据传递，对象标识，线程管理，调用过程控制等功能。**驱动层是整个Binder机制的核心**。
Framework层以驱动层为基础，提供了应用开发的基础设施。Framework层既包含了C++部分的实现，也包含了Java部分的实现。为了能将C++的实现复用到Java端，中间通过JNI进行衔接。
​

# 通信原理
Binder通信采用C/S架构，从组件视角来说，包含Client、Server、ServiceManager以及binder驱动，其中ServiceManager用于管理系统中的各种服务。架构图如下所示：
![avatar](https://cdn.nlark.com/yuque/0/2020/png/1759879/1601211334048-fa417b87-7211-4c46-b574-f7ed5cb7e4c4.png)
此处的Service Manager是指Native层的ServiceManager（C++），并非指framework层的ServiceManager(Java)。ServiceManager是整个Binder通信机制的大管家，是Android进程间通信机制Binder的守护进程。ServiceManager对于Binder Server的管理就好比车管所对于车牌号码的的管理，派出所对于身份证号码的管理：每个公开对外提供服务的Server都需要注册到ServiceManager中（通过addService），注册的时候需要指定一个唯一的id（这个id其实就是一个字符串）。Client要对Server发出请求，就必须知道服务端的id。Client需要先根据Server的id通过ServerManager拿到Server的标示（通过getService），然后通过这个标示与Server进行通信。
# 源码

- 驱动
```bash
https://android.googlesource.com/kernel/common/+/refs/tags/5.4-android11-0/drivers/android/

$ tree kernel/drivers/android/
├── Kconfig
├── Makefile
├── binder.c
├── binder_alloc.c
├── binder_alloc.h
├── binder_alloc_selftest.c
├── binder_internal.h
├── binder_trace.h
├── binderfs.c
└── vendor_hooks.c


https://android.googlesource.com/kernel/common/+/refs/tags/5.4-android11-0/include/uapi/linux/android/

$ tree kernel/include/uapi/linux/android/
├── binder.h
└── binderfs.h
```

- native
```bash
http://aospxref.com/android-11.0.0_r21/xref/frameworks/native/libs/binder/

$ tree frameworks/native/libs/binder/
├── Binder.cpp
├── BpBinder.cpp
├── IInterface.cpp
├── IPCThreadState.cpp
├── IServiceManager.cpp
├── Parcel.cpp
├── ProcessState.cpp
├── include
    ├── binder
        ├── Binder.h
        ├── BinderService.h
        ├── BpBinder.h
        ├── IBinder.h
        ├── IInterface.h
        ├── IPCThreadState.h
        ├── IServiceManager.h
        ├── ProcessState.h


http://aospxref.com/android-11.0.0_r21/xref/frameworks/native/cmds/servicemanager/

$ tree frameworks/native/cmds/servicemanager/
├── Access.cpp
├── Access.h
├── Android.bp
├── ServiceManager.cpp
├── ServiceManager.h
├── TEST_MAPPING
├── main.cpp
├── servicemanager.rc
├── test_sm.cpp
└── vndservicemanager.rc
```

- Java&jni
```bash
http://aospxref.com/android-11.0.0_r21/xref/frameworks/base/core/jni/

$ tree frameworks/base/core/jni/
├── AndroidRuntime.cpp
├── android_os_HidlMemory.cpp
├── android_os_HidlMemory.h
├── android_os_HidlSupport.cpp
├── android_os_HwBinder.cpp
├── android_os_HwBinder.h
├── android_os_HwParcel.cpp
├── android_os_HwParcel.h
├── android_os_HwRemoteBinder.cpp
├── android_os_HwRemoteBinder.h
├── android_os_Parcel.cpp
├── android_os_Parcel.h
├── android_util_Binder.cpp
├── android_util_Binder.h
├── android_util_Process.cpp
├── hwbinder
│   ├── EphemeralStorage.cpp
│   └── EphemeralStorage.h
├── include
    └── android_runtime
        ├── AndroidRuntime.h


http://aospxref.com/android-11.0.0_r21/xref/frameworks/base/core/java/android/os/

$ tree frameworks/base/core/java/android/os/
├── HidlMemory.java
├── HidlMemoryUtil.java
├── HidlSupport.java
├── HwBinder.java
├── HwBlob.java
├── HwParcel.java
├── HwRemoteBinder.java
├── IBinder.java
├── IHwBinder.java
├── IHwInterface.java
├── IInterface.java
├── Parcel.java
├── Parcelable.java
├── ServiceManager.java
├── ServiceManagerNative.java

```

# 文章列表
[Binder机制01-驱动](https://www.yuque.com/docs/share/33b2e243-6fed-4205-941e-5fc33bcd8da0?#)
Binder机制02-ServiceManager
03
04
05
...
后续补充