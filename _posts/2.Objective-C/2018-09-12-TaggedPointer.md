---
layout: post
title: "TaggedPointer"
date: 2018-09-12
description: "内存分布"
tag: Objective-C
---


## 目录
- [TaggedPointer](#content1)   
- [NSNumber](#content2)   
- [NSString](#content3)   
- [源码](#content4)   


<!-- ************************************************ -->
## <a id="content1">TaggedPointer</a>

从64bit开始，iOS引⼊了Tagged Pointer技术，用于优化 NSNumber、NSDate、NSString等小对象的存储。
在没有使⽤用Tagged Pointer之前， NSNumber等对象需要动态分配内存、维护引⽤计数等，NSNumber指针存储的是堆
中NSNumber对象的地址值

使用Tagged Pointer之后，NSNumber指针里⾯存储的数据变成了:Tag + Data，也就是将数据直接存储在了变量中。

当变量不够存储数据时，才会使用动态分配内存的方式来存储数据

objc_msgSend能识别Tagged Pointer，⽐如NSNumber的intValue方法，直接从指针提取数据，节省了了以前的调用开销

1 Tagged Pointer专⻔用来存储小的对象，例如NSNumber和NSDate

2 Tagged Pointer指针的值不再是地址了，而是真正的值。所以，实际上它不再 是一个对象了，它只是一个披着对象皮的普通变量而已。所以，它的内存并不存储 在堆中，也不需要malloc和free

3 在内存读取上有着3倍的效率，创建时比以前快106倍。

4 如何判断⼀个指针是否为Tagged Pointer?    
- iOS平台，最高有效位是1(第64bit)    
- Mac平台，最低有效位是1    

<img src="/images/underlying/other3.png" alt="img">


<!-- ************************************************ -->
## <a id="content2">NSNumber</a>


```objc
NSNumber * number1 = @1;
NSNumber * number2 = @2;
NSNumber * number3 = @0xffffffffffffffff;
NSLog(@"%p,%p,%p",number1,number2,number3);
```

日志打印
```objc
0xb000000000000012,0xb000000000000022,0x126d509e0
```

number1、number2数值较小采用Tagged Pointer技术，其值存储在变量地址中（指针中）。      
number3数值较大需要在堆上动态分配空间。变量number3内存储的是内存地址。      



<!-- ************************************************ -->
## <a id="content3">NSString</a>

```objc
NSString * name1 = [NSString stringWithFormat:@"abc"];
NSString * name2 = [NSString stringWithFormat:@"abcdefghijklmnopqist"];
NSLog(@"%p,%p",name1,name2);
```

日志打印
```objc
0xa000000006362613,0x156e63cb0
```
从日志可以看出name1采用了Tagged Pointer技术，name2是在堆上开辟了空间。


下面两段代码有什么区别会发生什么事情？

第一段
```objc
for (int i = 0; i<1000; i++) {
    dispatch_async(dispatch_get_global_queue(0, 0), ^{
        self.name = [NSString stringWithFormat:@"abc"];
    });
}
```

第二段
```objc
for (int i = 0; i<1000; i++) {
    dispatch_async(dispatch_get_global_queue(0, 0), ^{
        self.name = [NSString stringWithFormat:@"abcdefghijklmnopqist"];
    });
}
```

我们知道第一段采用Tagged Pointer技术，第二段在堆空间开辟释放空间。     
都开辟了子线程，存在线程安全问题。     

self.name = xxx;是调用了set方法，我们来看下set方法的本质

```objc
-(void)setName:(NSString *)name{
    if (_name != name) {
        [_name release];
        _name = [name retain];
    }
}
```

因为不停的对name赋值，在多线程访问的情况下[_name release];可能会被同时多次调用，释放已经释放的对象会引起坏内存访问。

毫无疑问第二段代码会引起程序的崩溃，第一段不存在内存释放所以程序会正常执行。

我们通过class来看下类型
```objc
self.name = [NSString stringWithFormat:@"abc"];

NSLog(@"%@",[self.name class]);

self.name = [NSString stringWithFormat:@"abcdefghijklmnopqist"];

NSLog(@"%@",[self.name class]);
```

日志打印如下
```objc
NSTaggedPointerString
__NSCFString
```

<!-- ************************************************ -->
## <a id="content4">源码</a>

可以在源码内搜索 _objc_makeTaggedPointer 查看taggedPointer相关代码


----------
>  行者常至，为者常成！


