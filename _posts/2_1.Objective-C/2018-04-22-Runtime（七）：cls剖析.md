---
layout: post
title: "Runtime（七）：cls剖析"
date: 2018-04-22
description: "Runtime（七）：cls剖析"
tag: Objective-C
---







## 目录
- [问题抛出](#content1)   
- [问题解析](#content2)   






<!-- ************************************************ -->
## <a id="content1"></a>问题抛出

先来看段代码

Person.h
```objc
NS_ASSUME_NONNULL_BEGIN

@interface Person : NSObject
@property (nonatomic, strong) NSString * name;

-(void)print;

@end
NS_ASSUME_NONNULL_END
```

Person.m
```objc
#import "Person.h"

@implementation Person

-(void)print{
    NSLog(@"my name is %@",self.name);
}
@end
```

ViewController.m
```objc
@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    
    id cls = [Person class];
    
    void * obj = &cls;
    
    [(__bridge id)obj print];
}
@end
```

print函数能不能调用成功？如果能调用成功打印结果是什么？

可以调用成功，打印结果如下
```objc
my name is <ViewController: 0x100704080>
```



<!-- ************************************************ -->
## <a id="content2"></a>问题解析

**一、为什么可以调用**

分析下上边代码的指针指向关系

<img src="/images/underlying/msgsend6.png" alt="img">

从指针的指向可以看出cls相当于isa指针，指向Person类对象。obj就可以认为是一个指向Person实例对象的指针。    
所以obj可以调用print方法。


**二、为什么打印结果是<ViewController: 0x100704080>**

1、如果是person对象调用print方法，self成员变量name是如何找到的？是在isa地址的基础上加8字节找到的。那么obj调用print的函数，相应找到的地址应该是<span style="color:red">cls的内存地址加8个字节</span>，那么这个地址对应的是什么内容？

2、要弄清问题1，就要说下iOS程序的内存分布：代码段  数据段  堆  栈  虚拟内存的地址从低到高

<img src="/images/underlying/msgsend7.png" alt="img">

堆空间分配地址从整体上来说由低到高，栈空间正好与它相反，地址的分配<span style="color:red">由高到低</span>。   


将ViewController.m文件通过以下指令编译并精简代码      
xcrun -sdk iphoneos clang -arch arm64 -rewrite-objc ViewController.m     

```objc
static void _I_ViewController_viewDidLoad(ViewController * self, SEL _cmd) {

    struct __rw_objc_super s = {
        (id)self,
        class_getSuperclass(objc_getClass("ViewController"))
    }
    
    objc_msgSendSuper)(a,sel_registerName("viewDidLoad"));

    
    id cls = objc_msgSend)(objc_getClass("Person"),sel_registerName("class"));

    void * obj = &cls;

   objc_msgSend)(obj, sel_registerName("print"));
}
```

在viewDidLoad方法内局部变量的分布如下

<img src="/images/underlying/msgsend8.png" alt="img">

可以看出cls的地址加8字节就是self，这就是为什么打印结果是<ViewController: 0x100704080>的原因。


**三、修改代码试下**
```objc
- (void)viewDidLoad {
    [super viewDidLoad];
    
    NSString * name = @"小明";
    
    id cls = [Person class];
    
    void * obj = &cls;
    
    [(__bridge id)obj print];
    
}
```

打印日志
```objc
 my name is 小明
```

可以按上边的逻辑分析下原因。

----------
>  行者常至，为者常成！


