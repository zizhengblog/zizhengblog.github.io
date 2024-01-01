---
layout: post
title: "Crash分析（一）：dSYM文件"
date: 2018-04-05
description: "Crash分析（一）：dSYM文件"
tag: Objective-C
---




- [参考文章：iOS 使用dSYM分析友盟错误日志](https://www.jianshu.com/p/adcf5ff5e5b2)
- [参考文章：iOS crash解析](https://www.jianshu.com/p/5e2455bdba38)


## 目录
- [什么是dSYM文件](#content1)   
- [dSYM文件的作用](#content2)   
- [dSYM文件的UUID](#content3)   




<!-- ************************************************ -->
## <a id="content1"></a>什么是dSYM文件

Xcode编译项目后，我们会看到一个同名的dSYM文件，dSYM是保存<span style="color:red">16进制函数地址映射信息</span>的中转文件（符号表），我们调试的symbols都会包含在这个文件中，并且每次编译项目的时候都会生成一个新的dSYM文件。

我们可以从两个地方找到dSYM文件：       
1、在release环境下，工程编译完成后，在下面目录下可以看到dSYM文件：           
`/Users/自己的用户名/Library/Developer/Xcode/DerivedData/Build/Products/Release-iphoneos`      

<img src="/images/underlying/other7.png" alt="img">

2、上传App Store时我们需要构建版本，该版本的APP会存在本地目录如下：       
`/Users/自己的用户名/Library/Developer/Xcode/Archives/2018-09-27/自己的appName 2018-9-27, 10.34 PM.xcarchive/dSYMs`    

<img src="/images/underlying/other8.png" alt="img">

3、在debug环境下默认是不会产生dSYM文件的，但通过环境配置也可以生成。    

`XCode -> Build Settings -> Code Generation -> Generate Debug Symbols -> Yes`

<img src="/images/underlying/other9.png" alt="img">

`XCode -> Build Settings -> Build Option -> Debug Information Format -> DWARF with dSYM File`

<img src="/images/underlying/other10.png" alt="img">

在debug环境下编译 

<img src="/images/underlying/other11.png" alt="img">





<!-- ************************************************ -->
## <a id="content2"></a>dSYM文件的作用

当我们开发完成将软件安装包提供给测试或上线App Store后，程序的崩溃报错就没法直观的通过xcode来了解。     
这个时候我们就需要分析crash report文件了，iOS设备中有日志文件，保存我们每个应用出错的函数内存地址。      
这个文件开发者可以通过几种方式拿到：xcode 、第三方工具、用户共享

这个时候我们就可以通过出错的函数地址去查询dSYM文件中程序对应的函数名和文件名。这也是为什么我们很有必要保存每个发布版本的 Archives文件了。






<!-- ************************************************ -->
## <a id="content3"></a>dSYM文件的UUID

每一个dSYM文件都有一个特定的UUID，并且与安装包的UUID对应（所以叫符号表），可以认为特定的安装包有一个唯一与之对应的dSYM文件。

如何获取dSYM文件的UUID？

终端输入指令：`xcrun dwarfdump --uuid appName.app.dSYM`

<img src="/images/underlying/other12.png" alt="img">

如何获取app的UUID？

终端输入指令：`xcrun dwarfdump --uuid appName.app/appName`

<img src="/images/underlying/other13.png" alt="img">

可以看到dSYM文件的UUID与APP的UUID在不同架构下是一一对应的。


----------
>  行者常至，为者常成！


