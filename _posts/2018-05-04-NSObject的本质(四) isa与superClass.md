---
layout: post
title: "NSObject的本质(四) isa与superClass"
date: 2018-05-04
description: "NSObject的本质"
tag: 底层原理
---


<h6>
  版权声明：本文为博主原创文章，未经博主允许不得转载。
  <a target="_blank" href="https://jianghuhike.github.io/1854.html">
  原文地址：https://jianghuhike.github.io/1854.html 
  </a>
</h6>




## 目录


- [isa](#content1)   
- [superClass](#content2)   
- [总结](#content3) 



<!-- ************************************************ -->
## <a id="content1"></a>isa
1.我们先把instance、 class、 meta-class放到一个表中看下

|instance 实例对象|class 类对象|meta-class 元类对象|
|isa|isa|isa|
|...|superClass|superClass|
|成员变量的值|属性/对象方法/协议/成员变量|类方法|
|...|...|...|

我们看到三种不同类型的对象中都有一个isa指针，那么它究竟是什么？    
先来看一张图     

<img src="/images/underlying/oc2.png" alt="img">

从图中可知：    
instance 的 isa指针 & ISA_MASK 的结果指向 class 。    
<span style="color:red">当调用对象方法时，instance通过isa指针找到class对象，最后找到 ***对象方法*** 的实现进行调用</span>    


class 的 isa指针 & ISA_MASK 的结果指向 meta-class 。     
<span style="color:red">当调用类方法时，class通过isa指针找到meta-class对象，最后找到 ***类方法*** 的实现进行调用</span>    

从64bit开始，isa需要进行一次位运算才能得到真实的地址值。
```objc
if __arm64__
  define ISA_MASK  0x0000000ffffffff8ULL
elif __X86_64__
  define ISA_MASK  0x00007ffffffffff8ULL
endif
```





<!-- ************************************************ -->
## <a id="content2"></a>superClass

我们假设现在有三个类Student、Person、NSObject;其继承关系是Student:Person:NSObject;    

当Student的instance对象要调用Person的对象方法时。

<img src="/images/underlying/oc3.png" alt="img">

<span style='color:red'>student->isa 找到 Student的class对象。</span>     
<span style='color:red'>Student的class对象->superClass  找到  Person的class对象   找到要调用的方法</span>

当Student的class对象要调用Person的类方法时。

<img src="/images/underlying/oc4.png" alt="img">

<span style='color:red'>student->isa 找到 Student的class对象 找到 Student的meta-class对象。</span>     
<span style='color:red'>Student的meta-class对象->superClass  找到  Person的meta-class对象   找到要调用的方法</span>



<!-- ************************************************ -->
## <a id="content3"></a>总结

还是先来看一张图：

<img src="/images/underlying/oc5.png" alt="img">

红色箭头显示了实例方法调用轨迹。    
绿色箭头显示了类方法调用轨迹。     

给NSObject添加Test分类,并注释掉 +(void)test;方法的实现

```objc
#import <Foundation/Foundation.h>
@interface NSObject (Test)
+ (void)test;
@end


@implementation NSObject (Test)
- (void)test{
    NSLog(@"+[NSObject test] - %p", self);
}

//+ (void)test{
//    NSLog(@"-[NSObject test] - %p", self);
//}
@end
```

创建Person类,对test方法只提供声明，注释掉实现，我们来看下调用结果
```objc
@interface MJPerson : NSObject

+ (void)test;

@end

@implementation MJPerson
//+ (void)test{
//    NSLog(@"+[MJPerson test] - %p", self);
//}
@end
```

我们来调用看下结果，
```objc
NSLog(@"[MJPerson class] - %p", [MJPerson class]);
NSLog(@"[NSObject class] - %p", [NSObject class]);
[Person test];
[NSObject test];
```

调用的打印结果如下：     
[Person class] - 0x1000011e0      
[NSObject class] - 0x7fff929c7140     
-[NSObject test] - 0x1000011e0     
-[NSObject test] - 0x7fff929c7140              

从调用结果可以看出，类方法的调用轨迹是循着绿色箭头的轨迹走的。    





----------
>  行者常至，为者常成！


