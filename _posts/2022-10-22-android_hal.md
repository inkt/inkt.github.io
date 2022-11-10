---
layout: post
title: Android HAL
tags: [Android]
author: laoYao
---

# Android硬件抽象层开发

![](/raw/android_structure.png)

## Android系统架构

* Linux内核   C语言

![](/raw/linux_driver.png)

```
通过驱动程序     将硬件    封装为Linux系统中的文件 
                对硬件寄存器的封装    为对文件的操作
```

* 硬件抽象层 C++/C

```
定义了一个标准接口供硬件供应商实现，

用来向下屏蔽硬件驱动模块的实现细节，向上提供统一的硬件访问服务

将对文件的操作 通过android提供的标准接口 封装为硬件模块hw_module_t与设备hw_device_t 
```

* Android框架层/硬件服务层  Java  <-- Jni --> C++/C

```
`硬件访问服务`通过`硬件抽象层模块`来为应用程序提供硬件读写操作接口

硬件抽象层模块是使用 C++/C 语言开发

应用程序框架中的硬件访问服务是使用JAVA语言开发

因此硬件访问服务需要通过JAVA本地接口(Java Native Interface JNI)来调用硬件抽象层模块的接口

通过jni将硬件模块hw_module_t与设备hw_device_t 中的C/C++方法映射为java可调用的本地方法
```

![](/raw/hal_1.png)

![](/raw/hal_2.png)

* 应用层 Java/Kotlin    

```
通过硬件访问服务，实现业务逻辑
```

## 硬件抽象层开发

```
Android系统为硬件抽象层中的模块接口定义了编写规范，Android系统的硬件抽象层以模块的形式来管理各个硬件访问接口，每一个硬件模块都对应有一个动态链接库文件。

每个硬件抽象层模块都使用结构体 hw_module_t 来描述，硬件设备则结构体 hw_device_t 来描述

一个硬件模块可有多个硬件设备
```

* 头文件


![](/raw/dot_h.png)


```
宏 FREG_HARDWARE_MODULE_ID 和 FREG_HARDWARE_DEVICE_ID 分别代表模块ID和设备ID

结构体freg_module_t用来描述自定义的模块结构体，他的第一个成员变量的类型是hw_module_t

结构体freg_device_t是自定义的设备结构体，他的第一个成员变量的类型是hw_device_t 用来描述我们的虚拟硬件设备，其中的fd代表文件描述符，表示需要打开的/dev/xxx 设备结点。其余两个函数指针用来读写 /dev/xxx 设备
```

* C++源文件

![](/raw/cpp.png)

```
HAL_MODULE_INFO_SYM 是模块变量，按照硬件抽象层的编写规范，每一个硬件抽象层模块必须导出一个名称为 HAL_MODULE_INFO_SYM 的符号，它指向一个自定义的硬件抽象层模块结构体，而且它的第一个类型为 hw_module_t 的成员变量

实例变量名必须为 HAL_MODULE_INFO_SYM ，tag也必须为 HARDWARE_MODULE_TAG ，这是Android硬件抽象层规范规定的。

一个硬件抽象层模块可能包含多个硬件设备，而这些硬件设备的打开操作都是由函freg_device_open来完成的，因此，函数freg_device_open会根据传递进来的参数id来判断要打开哪一个硬件设备。

在本利硬件抽象层模块freg中，只有一个虚拟硬件设备freg，它使用 freg_device_t来描述，因此，函数 freg_device_open 发现参数id与虚拟硬件设备freg的ID值匹配以后，就会分配一个 freg_device_t结构体，并对其成员变量进行初始化，

按照硬件抽象层模块的编写规范，硬件抽象层中的硬件设备标签 （dev->common.tag）必须设置为HARDWARE_DEVICE_TAG

编译后生成freg.default.so动态链接库文件
```

## jni

![](/raw/jni_1.png)

![](/raw/jni_2.png)

```
在函数 freg_init 中，首先通过Android硬件抽象层提供的 hw_module_get 函数来加载模块ID为 FREG_HARDWARE_MODULE_ID 的硬件抽象层模块，其中， FREG_HARDWARE_MODULE_ID 是在<hardware/freg.h>中定义的。Android硬件抽象层会根据 FREG_HARDWARE_MODULE_ID 的值在Android系统的/system/lib/hw目录中找到相应的模块，然后加载起来，并且返回hw_module_t接口给调用者使用。

文件中定义的 JNI方法表 method_table，分别将函数 freg_init、freg_setVal、freg_getVal的JNI方法注册为 init_native setVal_native getVal_native。文件最后调用了 jniRegisterNativeMethods()函数把JNI方法表注册到JAVA虚拟机中
```

## 框架层-硬件服务

```
Android系统的硬件访问服务通常运行在系统进程System中，而使用这些硬件访问服务的应用程序运行在其他的进程中，因此应用程序需要通过进程间通信机制来访问这些硬件访问服务，

Android系统提供了一种高效的进程间通信机制–Binder进程间通信机制，应用程序就是通过它来访问运行在系统进程System中的硬件访问服务的。

Binder进程间通信机制要求要求提供服务的一方必须实现一个具有跨进程访问能力的服务接口，以便使用服务的一方可以通过这个服务接口来访问它。
```

* 定义硬件访问服务接口 AIDL

![](/raw/aidl.png)

```
Android 系统提供了一种描述语言来定义具有跨进程访问能力的服务接口，这种描述语言称为 Android 接口描述语言(Android Interface Definition Language， AIDL)。

以AIDL定义的服务接口文件是以aidl为后缀的，在编译时，编译系统会将他们转换成Java文件，然后对他们进行编译，此处我们使用AIDL来定义硬件访问服务接口 IFregService。
```

* 实现硬件访问服务

![](/raw/service.png)

```
实现了IFregService接口的成员函数setVal和getVal

其中，成员函数setVal通过调用JNI方法来写虚拟硬件设备freg，

成员函数getVal调用JNI方法getVal_native来读虚拟硬件设备
```

* 注册硬件访问服务

![](/raw/service_2.png)

```
frameworks/base/service/java/com/android/server目录下，打开里面的 SystemService.java 文件，修改ServiceThread类的成员函数 run的实现, 注册硬件访问服务
```

## 应用层

```kotlin
View.setOnClickListener {
      // 调用硬件访问服务
      FregService.setVal (1)
}
```