---
layout: post
title: "OC速记"
date: 2023-02-10
tag: Overview
---





## 目录

- [1基础](#content1)  
- [2.1runtime](#content2)  
- [2.2多线程](#content3)   
- [2.3runloop](#content4)   
- [2.4自动释放池](#content5)  
- [2.5webview](#content6)  


<!-- ************************************************ -->
## <a id="content1">1基础</a>


**1、对象本质**    
实例对象、类对象、元类对象    
关系图(isa、super 指针)      

**2、kvc/kvo**   

kvc:先找方法，再找变量，都找不到报错    
kvo:派生类kvo_person,实例对象isa指向kvo_person, kvo_person的super指向Person类对象，kvo_person内重写setAge方法，  

**3、Category**       
+load方法、+initialize方法、关联对象(全局hash表)     

**4、block**       
block的定义和使用          
block的数据结构(impl+desc+捕获变量)和调用原理         
block的变量(全局变量、静态局部变量、自动变量)捕获原理(age/person)      
block的类型(globalBlock, stackBlock, mallocBlock; copy/dispose)     
_ _block的原理           
循环引用         

<!-- ************************************************ -->
## <a id="content2.1">2.1runtime</a>


**1、类对象的结构：isa、super、cache、bits**      

isa(这里说的是实例对象的isa)：   
是否优化、是否有关联对象、是否有弱引用、是否有引用计数表、引用计数 

super:     
指向父类    

cache:        
散列表，buket_t(key, _imp)           

bits:       
class_rw_t(方法列表，属性列表，协议列表)                
这些列表都是二维数组：method_list_t  -  method_t      

**2、消息机制**   

runtime主要指的是消息机制，有三个阶段：消息发送、动态方法解析、消息转发      

消息发送是在查找方法实现：当前类cache查找，方法列表查找，父类cache查找，父类方法列表查找      

动态方法解析是给类一个机会自己动态生成对应的方法       
调用resoleveInstanceMethod方法，在方法内给类添加方法实现，打标记，重走消息发送         

消息转发是当前类没有处理能力看看其它的类能不能处理              
forwardingTarget方法，返回一个能处理消息的实例对象         
methodSignature返回一个方法签名，如果返回空报doesntRecognizeSelector方法        
forwardInvocation 内可以进行一些自定义处理，比如进行一些提示      


**3、runtime的应用**   
修改 textField 的占位文字、字典转模型、自动归解档 （本质都是遍历属性列表，配合kvc完成）    

方法交换，解决数组越界崩溃问题    

消息转发，解决方法找不到的崩溃问题     

**4、其它**     
@synthesize 和 @dynamic       
super调用的原理      
isMemberOfClass 和 isKindOfClass      
cls分析     


<!-- ************************************************ -->
## <a id="content2.2">2.2多线程</a>


**1、进程与线程**   
进程资源分配的基本单位        
线程任务调度的基本单位     
1、提高执行效率，提高资源利用率     
2、线程本身也会占用资源，是程序变的复杂           

**2、gcd**   

串行队列，并行队列、同步任务、异步任务(死锁情况分析)   

线程组的两种用法：
dispatch_group_async     
dispatch_group_enter/dispatch_group_leave              

栅栏函数：  
dispatch_barrier_sync(queue, ^{ });    
dispatch_barrier_async(queue, ^{ });     

semaphore    
初始为0：可以设置依赖关系   
初始为1：可以设置串行，索的作用    

其它     
dispatch_once    
dispatch_after   

**3、perform**   
主线程：都依赖runloop        
子线程：依赖runloop的情况waitUntilDone:NO 和 afterDelay:      


**4、看看经典面试题**     


**5、NSOperation**         
对GCD的封装，异步并发队列        
最大并发数，依赖关系        


<!-- ************************************************ -->
## <a id="content2.3">2.3runloop</a>


**1、基础**  

什么是runloop,runloop的作用    
处理事件的运行循环，用来不停的处理输入事件(输入源和定时源)          
让线程在有工作的时候忙于工作，没工作的时候休眠         

运行在特定的mode下(defaultMode/trackingMode/commonMode)        
每个mode包含若干：source0/source1/observer/timer 

运行逻辑：运行顺序、如何唤醒、退出条件    

**2、相关**   

source1:基于port的线程间通讯，系统事件捕获，比如触摸事件    
timer:NSTimer/afterDelay     
gcd:回主线程        
source0:触摸事件、perform、通知(kvo)   

observer:    
runloop状态监听     
beforeWaiting:(UI刷新、手势回调、autoreleasepool)     

**3、重要**   

触摸事件是如何传递和响应的？        
屏幕 - springboard - source1 - source0 - handleEventQueue - UIWindow - SubView  
hitTest: 和 pointInside:方法          

UI刷新     
layoutSubViews调用时机(4个)          
setNeedsLayout 和 layoutIfNeeded 

drawRect调用时机(2个)        
setNeedsDisplay     

cpu负责计算、GPU负责渲染、垂直同步信号   


<!-- ************************************************ -->
## <a id="content2.4">2.4自动释放池</a>

放入自动释放池的对象叫autorelease对象，在离开作用域的时候不会立即释放，而是在合适的时机释放     
autoreleasePoolPage对象(双向链表)             
next、parent、child三个指针        
push方法和pop方法               

释放：系统释放(runloop)和手动释放     

autorelease对象

<!-- ************************************************ -->
## <a id="content2.5">2.5webview</a>

webview的两个协议：navigateDelegate/uiDelegate                   
webview与原生的通讯:evaluateJavascript:/messagehandler 和 iframe         
webkit:webcore和javascriptCore         
javascriptCore:jsvm/jscontext/jsvalue/jsexport            









----------
>  行者常至，为者常成！


