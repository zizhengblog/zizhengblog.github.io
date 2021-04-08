---
layout: post
title: "Crash分析（三）：crash文件符号化"
date: 2018-10-21
description: "Crash分析（三）：crash文件符号化"
tag: Objective-C
---




- [参考文章：iOS实录14：浅谈iOS Crash（一）](https://www.jianshu.com/p/3261493e6d9e)
- [参考文章：crash文件符号化攻略](https://www.jianshu.com/p/8cac0b87ade2)
- [参考文章：Xcode崩溃日志分析工具symbolicatecrash用法](https://www.jianshu.com/p/e428501ff278)


## 目录
- [文件准备与验证](#content1)   
- [符号化](#content2)   
- [atos简单使用](#content3)   
- [symbolicatecrash使用](#content4)   






<!-- ************************************************ -->
## <a id="content1"></a>文件准备与验证

符号化需要有dSYM（符号表）文件，crash文件。

我们创建一个Person类，在.h文件内声明`-(void)test;`方法，但并不实现。

执行下面代码
```objc
Person * person = [[Person alloc] init];
[person test];
```
将测试demo编译运行安装到手机，在相应目录下取出dSYM文件保存。            
手机在非调试状态下运行demo，才会生成crash文件，然后导出crash文件。

执行指令查看dSYM文件的UUID
```
xcrun dwarfdump --uuid iOSTest.app.dSYM

UUID: C9C138E6-4CF7-3790-9D07-BD44682717E1 (armv7) iOSTest.app.dSYM/Contents/Resources/DWARF/iOSTest
UUID: F1915EB2-AEE0-3EDD-AE99-08F7997CDF77 (arm64) iOSTest.app.dSYM/Contents/Resources/DWARF/iOSTest
```

执行指令查看crash文件的UUID

```
grep --after-context=2 "Binary Images:" iOSTest.crash

Binary Images:
0x100084000 - 0x10008bfff iOSTest arm64  <f1915eb2aee03eddae9908f7997cdf77> /var/mobile/Containers/Bundle/Application/48885D93-C3A2-4CAA-BC40-812E7307DECA/iOSTest.app/iOSTest
```

可以看到arm64架构下      
dSYM文件的UUID ：F1915EB2-AEE0-3EDD-AE99-08F7997CDF77      
crash文件的UUID：f1915eb2aee03eddae9908f7997cdf77      
对比一致，也就是说crash文件能够通过dSYM将地址信息符号化。       



<!-- ************************************************ -->
## <a id="content2"></a>符号化


crash文件线程回溯部分信息如下
```objc
Last Exception Backtrace:
0   CoreFoundation                	0x181a75900 __exceptionPreprocess + 124
1   libobjc.A.dylib               	0x1810e3f80 objc_exception_throw + 55
2   CoreFoundation                	0x181a7c61c -[NSObject+ 1230364 (NSObject) doesNotRecognizeSelector:] + 211
3   CoreFoundation                	0x181a795b8 ___forwarding___ + 871
4   CoreFoundation                	0x18197d68c _CF_forwarding_prep_0 + 91
5   iOSTest                       	0x100089170 0x100084000 + 20848
6   UIKit                         	0x1867680c0 -[UIViewController loadViewIfRequired] + 995
7   UIKit                         	0x186767cc4 -[UIViewController view] + 27
8   UIKit                         	0x18676eab4 -[UIWindow addRootViewControllerViewIfPossible] + 75
9   UIKit                         	0x18676bfa0 -[UIWindow _setHidden:forced:] + 251
10  UIKit                         	0x1867e1cd0 -[UIWindow makeKeyAndVisible] + 47
11  UIKit                         	0x186a0c358 -[UIApplication _callInitializationDelegatesForMainScene:transitionContext:] + 3455
12  UIKit                         	0x186a104b8 -[UIApplication _runWithMainScene:transitionContext:completion:] + 1671
13  UIKit                         	0x186a0d5c0 -[UIApplication workspaceDidEndTransaction:] + 167
14  FrontBoardServices            	0x18302b790 -[FBSSerialQueue _performNext] + 183
15  FrontBoardServices            	0x18302bb10 -[FBSSerialQueue _performNextFromRunLoopSource] + 55
16  CoreFoundation                	0x181a2cefc __CFRUNLOOP_IS_CALLING_OUT_TO_A_SOURCE0_PERFORM_FUNCTION__ + 23
17  CoreFoundation                	0x181a2c990 __CFRunLoopDoSources0 + 539
18  CoreFoundation                	0x181a2a690 __CFRunLoopRun + 723
19  CoreFoundation                	0x181959680 CFRunLoopRunSpecific + 383
20  UIKit                         	0x1867d6580 -[UIApplication _run] + 459
21  UIKit                         	0x1867d0d90 UIApplicationMain + 203
22  iOSTest                       	0x100089c84 0x100084000 + 23684
23  libdyld.dylib                 	0x1814fa8b8 start + 3
```

可以看出行号5、行号22的堆栈信息并不能看出具体是哪里出现问题。

|帧编号<br>数字越小，发生时间越晚|二进制库的名称|调用方法的地址|基本地址+偏移量<br>调用方法地址=基本地址+偏移量|
|5|   iOSTest|                       	0x100089170| 0x100084000 + 20848|
|22|  iOSTest|                       	0x100089c84| 0x100084000 + 23684|

应用崩溃发生在运行时地址0x100089170，该进程的运行时起始地址是0x100084000，崩溃处距离进程起始地址的偏移量为十进制的20848(对应十六进制为0x5170)。三者对应关系：      
0x100089170 = 0x100084000 + 0x5170

崩溃堆栈中的起始地址和崩溃地址均为运行时地址，根据虚拟内存偏移量不变原理，只要提供了符号表TEXT段的起始地址，再加上偏移量（这里为0x5170）就能得到符号表中的堆栈地址，即：      
`符号表堆栈地址 = 符号表起始地址 + 偏移量`


<!-- ************************************************ -->
## <a id="content3"></a>atos简单使用


开发者从crash文件能获取到错误堆栈信息，而使用atos工具就是把地址对应的具体符号信息找到。       
它是一个可以把地址转换为函数名（包括行号）的工具。        

atos无需计算崩溃地址对应的符号表地址的方式，命令格式如下：

```
atos -o Your.app.dSYM/Contents/Resources/DWARF/Your -arch armv7 -l 进程起始地址 崩溃发生在运行时地址
```

帧编号5栈信息中地址符号化：
```
atos -o iOSTest.app.dSYM/Contents/Resources/DWARF/iOSTest -arch arm64 -l 0x100084000 0x100089170

-[ViewController viewDidLoad] (in iOSTest) (ViewController.mm:65)
```

帧编号22栈信息中地址符号化：
```
//可以进入到 iOSTest.app.dSYM/Contents/Resources/DWARF 目录下
atos -o iOSTest -arch arm64 -l 0x100084000 0x100089c84

main (in iOSTest) (main.m:20)
```

<!-- ************************************************ -->
## <a id="content4"></a>symbolicatecrash使用

symbolicatecrash是Xcode自带的一个分析工具，可以通过机器上的崩溃日志和应用的.dSYM文件定位发生崩溃的位置，把crash日志中的一堆地址替换成代码相应位置

1、在你的MAC桌面创建一个新文件夹，并且命名为"CrashReport"

2、查看symbolicatecrash文件路径：     
打开终端输入：`find /Applications/Xcode.app -name symbolicatecrash -type f`

```
find /Applications/Xcode.app -name symbolicatecrash -type f
/Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/iOSSupport/Library/PrivateFrameworks/DVTFoundation.framework/Versions/A/Resources/symbolicatecrash
/Applications/Xcode.app/Contents/Developer/Platforms/WatchSimulator.platform/Developer/Library/PrivateFrameworks/DVTFoundation.framework/symbolicatecrash
/Applications/Xcode.app/Contents/Developer/Platforms/AppleTVSimulator.platform/Developer/Library/PrivateFrameworks/DVTFoundation.framework/symbolicatecrash
/Applications/Xcode.app/Contents/Developer/Platforms/iPhoneSimulator.platform/Developer/Library/PrivateFrameworks/DVTFoundation.framework/symbolicatecrash
/Applications/Xcode.app/Contents/SharedFrameworks/DVTFoundation.framework/Versions/A/Resources/symbolicatecrash
```
文件路径是：`/Applications/Xcode.app/Contents/SharedFrameworks/DVTFoundation.framework/Versions/A/Resources/symbolicatecrash`

将symbolicatecrash拷贝到CrashReport文件夹

3、将dSYM文件与crash文件拷贝到CrashReport文件夹

4、依次执行下面指令生成iOSTestSymbol.crash

`cd /Users/username/Desktop/CrashReport`

`export DEVELOPER_DIR=/Applications/XCode.app/Contents/Developer`

`./symbolicatecrash ./iOSTest.crash ./iOSTest.app.dSYM > iOSTestSymbol.crash`

看下iOSTestSymbol.crash文件内线程回溯部分
```objc
Last Exception Backtrace:
0   CoreFoundation                	0x181a75900 __exceptionPreprocess + 124
1   libobjc.A.dylib               	0x1810e3f80 objc_exception_throw + 55
2   CoreFoundation                	0x181a7c61c -[NSObject+ 1230364 (NSObject) doesNotRecognizeSelector:] + 211
3   CoreFoundation                	0x181a795b8 ___forwarding___ + 871
4   CoreFoundation                	0x18197d68c _CF_forwarding_prep_0 + 91
5   iOSTest                       	0x100089170 -[ViewController viewDidLoad] + 20848 (ViewController.mm:65)
6   UIKit                         	0x1867680c0 -[UIViewController loadViewIfRequired] + 995
7   UIKit                         	0x186767cc4 -[UIViewController view] + 27
8   UIKit                         	0x18676eab4 -[UIWindow addRootViewControllerViewIfPossible] + 75
9   UIKit                         	0x18676bfa0 -[UIWindow _setHidden:forced:] + 251
10  UIKit                         	0x1867e1cd0 -[UIWindow makeKeyAndVisible] + 47
11  UIKit                         	0x186a0c358 -[UIApplication _callInitializationDelegatesForMainScene:transitionContext:] + 3455
12  UIKit                         	0x186a104b8 -[UIApplication _runWithMainScene:transitionContext:completion:] + 1671
13  UIKit                         	0x186a0d5c0 -[UIApplication workspaceDidEndTransaction:] + 167
14  FrontBoardServices            	0x18302b790 -[FBSSerialQueue _performNext] + 183
15  FrontBoardServices            	0x18302bb10 -[FBSSerialQueue _performNextFromRunLoopSource] + 55
16  CoreFoundation                	0x181a2cefc __CFRUNLOOP_IS_CALLING_OUT_TO_A_SOURCE0_PERFORM_FUNCTION__ + 23
17  CoreFoundation                	0x181a2c990 __CFRunLoopDoSources0 + 539
18  CoreFoundation                	0x181a2a690 __CFRunLoopRun + 723
19  CoreFoundation                	0x181959680 CFRunLoopRunSpecific + 383
20  UIKit                         	0x1867d6580 -[UIApplication _run] + 459
21  UIKit                         	0x1867d0d90 UIApplicationMain + 203
22  iOSTest                       	0x100089c84 main + 23684 (main.m:20)
23  libdyld.dylib                 	0x1814fa8b8 start + 3
```

发现帧编号5、帧编号22对应的信息已经符号化。


----------
>  行者常至，为者常成！


