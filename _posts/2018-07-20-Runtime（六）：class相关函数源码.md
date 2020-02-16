---
layout: post
title: "Runtime（六）：class相关函数源码"
date: 2018-07-20
description: "Runtime（六）：class相关函数源码"
tag: 底层原理
---


<h6>
  版权声明：本文为博主原创文章，未经博主允许不得转载。
  <a target="_blank" href="https://jianghuhike.github.io/18720.html">
  原文地址：https://jianghuhike.github.io/18720.html 
  </a>
</h6>




## 目录
- [class函数](#content1)   
- [superclass函数](#content2)   
- [isMemberOfClass函数](#content3)   





<!-- ************************************************ -->
## <a id="content1"></a>class函数

先看下源码实现

```objc
- (Class)class {
    return object_getClass(self);
}

+ (Class)class {
    return self;
}
```

对象方法调用时通过object_getClass函数返回的是类对象。
类方法调用时直接返回self，即将自身返回也就是类对象。       
所以不管对象方法class还是类方法class返回的都是类对象。      

```
NSObject * obj = [[NSObject alloc] init];

//不管调用多少次class返回的都是类对象
[[[obj class] class] class];

[obj class];//返回类对象，之后调用class都返回自身
```



<!-- ************************************************ -->
## <a id="content2"></a>superclass函数

先看下源码实现

```
- (Class)superclass {
    return [self class]->superclass;
}

+ (Class)superclass {
    return self->superclass;
}

```

与class方法类似，对象方法superclass和类方法superclass返回的结果一样，都是父类的类对象。

不同的是一直调用superclass会一直向上找父类的父类
```
//Student:Person:NSObject

Student * stu = [[Student alloc] init];
[[stu superclass] superclass]//NSObject
```


<!-- ************************************************ -->
## <a id="content3"></a>isMemberOfClass

先看下源码实现

```objc
- (BOOL)isMemberOfClass:(Class)cls {
    return [self class] == cls;
}

+ (BOOL)isMemberOfClass:(Class)cls {
    return object_getClass((id)self) == cls;
}
```

对象方法：是拿类对象与cls对比是否相等。
类方法：是拿元类对象与cls对比是否相等。    

先看下对象方法的示例

```
//Person:NSObject
Person * person = [[Person alloc] init];

//cls:Person
//[self class]:Person
//Person == Person所以打印结果为1
NSLog(@"%d",[person isMemberOfClass:[Person class]]);//1

//cls:NSObject
//[self class]:Person
//Person !=NSObject 所以打印结果为0
NSLog(@"%d",[person isMemberOfClass:[NSObject class]]);//0
```

再看下类方法的示例

```
//cls:Person
//object_getClass((id)self):Meta-Person
//Meta-Person != Person 所以打印结果为0
NSLog(@"%d",[Person isMemberOfClass:[Person class]]);//0

//cls:NSObject
//object_getClass((id)self):Meta-Person
//Meta-Person != NSObject 所以打印结果为0
NSLog(@"%d",[Person isMemberOfClass:[NSObject class]]);//0

//cls:Meta-Person
//object_getClass((id)self):Meta-Person
//Meta-Person == Meta-Person 所以打印结果为1
NSLog(@"%d",[Person isMemberOfClass:object_getClass([Person class])]);//1
```









----------
>  行者常至，为者常成！


