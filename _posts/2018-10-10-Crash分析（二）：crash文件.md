---
layout: post
title: "Crash分析（二）：crash文件"
date: 2018-10-10
description: "Crash分析（二）：crash文件"
tag: 底层原理
---


<h6>
  版权声明：本文为博主原创文章，未经博主允许不得转载。
  <a target="_blank" href="https://jianghuhike.github.io/181010.html">
  原文地址：https://jianghuhike.github.io/181010.html 
  </a>
</h6>

<!-- - [参考文章：iOS 使用dsym分析友盟错误日志](https://www.jianshu.com/p/adcf5ff5e5b2) -->


## 目录
- [什么是crash文件](#content1)   
- [crash文件的结构](#content2)   
- [crash文件的UUID](#content3)   




<!-- ************************************************ -->
## <a id="content1"></a>什么是crash文件

当程序运行崩溃的时候，系统会把运行的最后时刻的运行信息记录下来，存储到一个文件中，这个文件就是Crash文件。

如何获得crash文件？

1、将出错的手机连接Xcode，xcode->Window->Devices and Simulators -> View Device Logs 将文件导出

<img src="/images/underlying/crash0.png" alt="img">


2、第三方的工具导出。如：itools

3、通过用户与开发者共享：设置->隐私->分析->共享iPhone分析

<img src="/images/underlying/crash1.png" alt="img">


<!-- ************************************************ -->
## <a id="content2"></a>crash文件的结构


```objc
//1、进程信息
Hardware Model: iPhone9,2
Process: AppName [3580]
Path: /var/containers/Bundle/Application/C7B90C8A-E269-4413-A011-552971D1ED39/AppName.app
Identifier: xxxx.xxx.xxxx.xxx
Version: xx.xx
Code Type: ARM-64 (Native)
Parent Process:  [1]

//2、基本信息
Date/Time: 2017-05-22 03:05:06.743 +0800
OS Version: iPhone OS 10.2.1 (14D27)

//3、异常信息
Exception Type: NSInvalidArgumentException(SIGABRT)
Exception Codes: -[NSNull integerValue]: unrecognized selector sent to instance 0x1a9d88ef8 at 0x00000001835c7014
Crashed Thread: 0

//4、线程回溯 （展示发生Crash线程的回溯信息，其他略）
Thread 0 Crashed: 
0  libsystem_kernel.dylib         0x00000001835c7014 __pthread_kill + 4
1  libsystem_c.dylib              0x000000018353b400 abort + 140
2  AppName                         0x0000000100a26704 0x0000000100028000 + 10479360
3  CoreFoundation                 0x00000001845f9538 ___handleUncaughtException +  644
2  CoreFoundation                 0x0000000184600268 ___methodDescriptionForSelector
3  CoreFoundation                 0x00000001845fd270 ____forwarding___ +  916
4  CoreFoundation                 0x00000001844f680c _CF_forwarding_prep_0 + 80
5  AppName                         0x0000000100205280 0x0000000100028000 + 1954432
6  AppName                         0x00000001002ae59c 0x0000000100028000 + 2647440
7  AppName                         0x0000000100482944 0x0000000100028000 + 4565312
16 CoreFoundation                 0x00000001845a6810 ___CFRUNLOOP_IS_SERVICING_THE_MAIN_DISPATCH_QUEUE__ +  12
     +  12
17 CoreFoundation                 0x00000001845a43fc ___CFRunLoopRun +  1660
18 CoreFoundation                 0x00000001844d22b8 CFRunLoopRunSpecific + 436

//5、进程状态（展示部分）
Thread 0 crashed with ARM 64 Thread State:
     x0:  000000000000000000    x1: 000000000000000000    x2: 000000000000000000     x3: 0xffffffffffffffff
     x4:  0x0000000000000010    x5: 0x0000000000000020    x6: 000000000000000000     x7: 000000000000000000
     x8:  0x0000000008000000    x9: 0x0000000004000000   x10: 000000000000000000    x11: 0x00000001ac336c83
    x12: 0x00000001ac336c83    x13: 0x0000000000000018   x14: 0x0000000000000001    x15: 0x0000000000000881
    x16: 0x0000000000000148    x17: 000000000000000000   x18: 000000000000000000    x19: 0x0000000000000006

//6、二进制映像 （展示部分）
Binary Images:
0x100028000 - 0x1011dbfff +AppName arm64 <ff7a4009322b386ea3e8e9a3fde05be4> /var/containers/Bundle/Application/C7B90C8A-E269-4413-A011-552971D1ED39/AppName.app/AppName
0x18368a000 - 0x183693fff  libsystem_pthread.dylib arm64 <258dc0c51499393bba7ba3e83dc5bfbb> /usr/lib/system/libsystem_pthread.dylib
0x1835a8000 - 0x1835ccfff  libsystem_kernel.dylib arm64 <1baa3f5629c43467879d4cf463a20b06> /usr/lib/system/libsystem_kernel.dylib
0x1834b1000 - 0x1834b5fff  libdyld.dylib arm64 <db54f120486a3710a684ce8bb1cb9d71> /usr/lib/system/libdyld.dylib
0x1834d8000 - 0x183556fff  libsystem_c.dylib arm64 <8a5a190d70563f3c8d4ce16cab74f599> /usr/lib/system/libsystem_c.dylib
0x183481000 - 0x1834b0fff  libdispatch.dylib arm64 <fb1d0baf642337d1bea0af309586df97> /usr/lib/system/libdispatch.dylib
0x183028000 - 0x183401fff  libobjc.A.dylib arm64 <538f809dcd7c35ceb59d99802248f045> /usr/lib/libobjc.A.dylib
```

<span style="color:red">crash文件中大部分是十六进制的内存地址，对于开发者而言定位问题很不直观。要想看到跟开发时控制台输出那样直观的信息，需要对crash文件进行符号化，需要用到dSYM（符号表）文件。</span>

<!-- ************************************************ -->
## <a id="content3"></a>crash文件的UUID

```
//6、二进制映像 （展示部分）
Binary Images:
0x100028000 - 0x1011dbfff +AppName arm64 <ff7a4009322b386ea3e8e9a3fde05be4> /var/containers/Bundle/Application/C7B90C8A-E269-4413-A011-552971D1ED39/AppName.app/AppName
```

ff7a4009322b386ea3e8e9a3fde05be4       
就是crash文件的UUID,该UUID与dSYM文件、app的UUID保持一致，才能代表是某一个具体的app安装包产生的crash;



----------
>  行者常至，为者常成！


