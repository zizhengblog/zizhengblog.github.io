---
layout: post
title: "Runtime（五）：super详解"
date: 2018-07-18
description: "Runtime（五）：super详解"
tag: 底层原理
---







## 目录
- [super](#content1)   
- [经典示例](#content2)   



<!-- ************************************************ -->
## <a id="content1"></a>super

看段代码
```objc
#import "Student.h"

@implementation Student
-(void)test{
    [super test];
}

@end
```

通过指令：`xcrun -sdk iphoneos clang -arch arm64 -rewrite-objc Student.m`并精简代码

Student.cpp文件

```objc
struct __rw_objc_super {
    struct objc_object *object;
    struct objc_object *superClass;
};


static void _I_Student_test(Student * self, SEL _cmd) {
    
    struct __rw_objc_super arg = {
        self,
        class_getSuperclass(objc_getClass("Student"))
    }
    
    objc_msgSendSuper(arg,sel_registerName("test"));
}

```

可以看到 `[super test];` 在底层转为了 `objc_msgSendSuper(argu1,argu2);`              
argu1是结构体__rw_objc_super类型的变量，其中有两个成员，self、[self superclass];     
argu2是消息名称@selector(test);    

也就是说在消息发送阶段相当于传递了三个数据：      
self :消息的接收守者（receiver）<span style="color:red">虽然我们看着是super在调用，但实际上还是self在调用</span>     
[self superclass]:<span style="color:red">查找实例方法时跳过当前的类对象，从父类的类对象里开始查找</span>      
@selector(test):消息名称，被查找的方法名。     


<!-- ************************************************ -->
## <a id="content2"></a>经典示例


```objc
#import "Student.h"

@implementation Student
- (instancetype)init{
    NSLog(@"[person class]     = %@",[self class]);
    NSLog(@"[self superclass]  = %@",[self superclass]);
    NSLog(@"[super  class]     = %@",[super  class]);
    NSLog(@"[super superclass] = %@",[super superclass]);

    return nil;
}
```

我们了解了super的本质后，上边代码的打印结果是什么？

在回答之前，先来看下class跟superclass的源码实现

```
- (Class)class {
    return object_getClass(self);
}

- (Class)superclass {
    return [self class]->superclass;
}
```

再结合super的本质分析，我们不难得出结果
```objc
[person class]     = Student
[self superclass]  = Person
[super  class]     = Student
[super superclass] = Person
```









----------
>  行者常至，为者常成！


