---
layout: post
title: "NSObject的本质(三) 对象分类"
date: 2018-03-03
description: "NSObject的本质"
tag: Objective-C
---




## 目录


* [对象分类](#content1)



<!-- ************************************************ -->
## <a id="content2"></a>对象分类
**一、OC对象主要分为三种**    
- instance    实例对象
- class       类对象
- meta-class  元类对象

**二、instance**   
```objc
Person * person1 = [[Person alloc] init];
person1->_age = 3;

Person * person2 = [[Person alloc] init];
person2->_age = 4;
```

person1与person2是NSObject的instance对象(实例对象)，它们是两个不同的对象，分别占据着两块不同的内存。      
instance对象在内存中存储的信息包括：       
isa指针   
其它成员变量的值   

person1<Person: 0x280d80c30>的内存分布情况：    

|instance|地址|存储值|
|isa|0x0000000280d80c30|0x000001a1047655a9 Person|
|_age|0x0000000280d80c38|3|

person2<Person: 0x280d80c40>的内存分布情况：    

|instance|地址|存储值|
|isa|0x0000000280d80c40|0x000001a1047655a9 Person|
|_age|0x0000000280d80c48|4|

由上可知，person1与person2占用不同的内存，person1与person2的isa指向的是同一个地址。这个地址就是类对象的地址。

**三、class**    
```objc
NSObject *object1 = [[NSObject alloc] init];
NSObject *object2 = [[NSObject alloc] init];

Class objectClass1 = [object1 class];
Class objectClass2 = [object2 class];
Class objectClass3 = object_getClass(object1);
Class objectClass4 = object_getClass(object2);
Class objectClass5 = [NSObject class];

NSLog(@"%p %p",
      object1,
      object2);

NSLog(@"%p %p %p %p %p",
      objectClass1,
      objectClass2,
      objectClass3,
      objectClass4,
      objectClass5);
```
打印结果为：     
0x100535d00 0x100536120      
0x7fff929c7140 0x7fff929c7140 0x7fff929c7140 0x7fff929c7140 0x7fff929c7140     
objectClass1 ~ objectClass5都是NSObject的class对象（类对象）。    
他们是同一个对象，每个类在内存中有且只有一个类对象。   

instance与class对象在内存中存储信息对比：

|instance 实例对象|class 类对象|
|isa|isa|
|成员变量的值|superClass|
|成员变量的值|属性信息|
|成员变量的值|对象方法信息|
|成员变量的值|协议信息|
|成员变量的值|成员变量信息|
|...|...|

另外实例对象的isa指针（这里需要与一个mask做一次运算）是指向类对象的。   


**四、meta-class**    
1.先来看几个函数
```
//这两个方法都是返回类对象
 - (Class)class;
 - (Class)class{
     return self->isa;
 }
 
 + (Class)class;
 + (Class)class{
     return self;
 }
```

看下面的例子
```objc
//知道了实现，就能明白不论调用几次都是返回类对象
Class obj1 = [NSObject class];
Class obj2 = [[NSObject class] class];
Class obj3 = [[[NSObject class] class] class];

//obj1 obj2 obj3 都指向同一个对象，类对象NSObject
```

runtime API
```
#import <objc/runtime.h>
objc_getClass(const char * _Nonnull name);
1> 传入字符串类名
2> 返回对应的类对象

object_getClass(id  _Nullable obj);
1> 传入的obj可能是instance对象、class对象、meta-class对象
2> 返回值
a) 如果是instance对象，返回class对象
b) 如果是class对象，返回meta-class对象
c) 如果是meta-class对象，返回NSObject（基类）的meta-class对象
```

2.meta-class对象

获取meta-class对象
```objc
//instance 实例对象  0x000000010079f0c0
NSObject *object = [[NSObject alloc] init];

//class 类对象   0x00007fff929c7140
Class objectClass = [object1 class];

//meta-class对象，元类对象  0x00007fff929c70f0
//将类对象当做参数传入，获得元类对象
Class objectMetaClass = object_getClass(objectClass);
```
objectMetaClass是NSObject的meta-class对象，元类对象。每个类在内存中有且只有一个meta-class对象。       
meta-class对象和class对象的内存结构是一样的，但是用途不一样，在内存中存储的信息主要包括：

|meta-class 元类对象|
|isa|
|superClass|
|类的方法信息(class method)|
|...|

查看是否是元类对象
```objc
class_isMetaClass(Class  _Nullable __unsafe_unretained cls);
```

```objc
BOOL isMeta = class_isMetaClass(object_getClass([NSObject class]));
NSLog(@"isMeta=%d",isMeta);//isMeta=1

BOOL isMeta1 = class_isMetaClass([NSObject class]);
NSLog(@"isMeta1=%d",isMeta1);//isMeta=0
```




----------
>  行者常至，为者常成！


