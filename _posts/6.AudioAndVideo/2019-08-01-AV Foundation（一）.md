---
layout: post
title: "1、AV Foundation之视频捕捉-关键概念"
date: 2019-08-01
description: ""
tag: 音视频
---


<h6>版权声明：本文为博主原创文章，未经博主允许不得转载。</h6>


- [文章摘抄自：https://www.jianshu.com/p/8c5c150dba65，用于记录学习](https://www.jianshu.com/p/8c5c150dba65)


## 目录

* [捕捉功能综述](#content1)



<!-- ************************************************ -->
## <a id="content1"></a>捕捉功能综述

AV Foundation 照片和视频捕捉功能是从框架搭建之初就是它的强项。 从iOS 4.0 我们就可以直接访问iOS的摄像头和摄像头生成的数据（照片、视频）。目前捕捉功能仍然是苹果公司媒体工程师最关注的领域。 

核心的捕捉类在iOS 和 OS X上是一致的。除了Mac OSX 为截屏功能定义了AVCaptureScreenInput 类。但iOS上由于沙盒的限制不提供该类。我们讨论的大部分功能都适应于OS X开发的。想尝试Mac开发的同学，可以挑战一下哦！


**一、捕捉会话**

AV Foundation  捕捉栈核心类是AVCaptureSession。一个捕捉会话相当于一个虚拟的“插线板”。用于连接输入和输出的资源。

**二、捕捉设备**

AVCaptureDevice为摄像头、麦克风等物理设备提供接口。大部分我们使用的设备都是内置于MAC或者iPhone、iPad上的。当然也可能出现外部设备。但是AVCaptureDevice 针对物理设备提供了大量的控制方法。比如控制摄像头聚焦、曝光、白平衡、闪光灯等。

**三、捕捉设备的输入**

注意：为捕捉设备添加输入，不能添加到AVCaptureSession 中，必须通过将它封装到一个AVCaptureDeviceInputs实例中。这个对象在设备输出数据和捕捉会话间扮演接线板的作用。

**四、捕捉的输出**

AVCaptureOutput 是一个抽象类。用于为捕捉会话得到的数据寻找输出的目的地。框架定义了一些抽象类的高级扩展类。例如 AVCaptureStillImageOutput 和 AVCaptureMovieFileOutput类。使用它们来捕捉静态照片、视频。例如 AVCaptureAudioDataOutput 和 AVCaptureVideoDataOutput ,使用它们来直接访问硬件捕捉到的数字样本。

**五、捕捉连接**

AVCaptureConnection类.捕捉会话先确定由给定捕捉设备输入渲染的媒体类型，并自动建立其到能够接收该媒体类型的捕捉输出端的连接。

**六、捕捉预览**

如果不能在影像捕捉中看到正在捕捉的场景，那么应用程序用户体验就会很差。幸运的是框架定义了AVCaptureVideoPreviewLayer 类来满足该需求。这样就可以对捕捉的数据进行实时预览。






----------
>  行者常至，为者常成！


