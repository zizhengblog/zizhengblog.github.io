---
layout: post
title: "Runtime（四）：@synthesize与@dynamic"
date: 2018-07-15
description: "Runtime（四）：@synthesize与@dynamic"
tag: 底层原理
---


<h6>
  版权声明：本文为博主原创文章，未经博主允许不得转载。
  <a target="_blank" href="https://jianghuhike.github.io/18715.html">
  原文地址：https://jianghuhike.github.io/18715.html 
  </a>
</h6>




## 目录
- [@property](#content1)   
- [@synthesize](#conten2)   
- [@dynamic](#content3)   

<!-- ************************************************ -->
## <a id="content1"></a>@property


在说这两个关键字之前我们先来看下@property关键字

创建一个Person类

Person.h文件
```
@interface Person : NSObject;
@property (nonatomic, assign) int age;
@end
```

Person.m文件
```objc
#import "Person.h"
@implementation Person

@end
```

通过指令`xcrun -sdk iphoneos clang -arch arm64 -rewrite-objc Person.m`     
生成Person.cpp文件

```objc
struct Person_IMPL {
	struct NSObject_IMPL NSObject_IVARS;
	int _age;
};

static int _I_Person_age(Person * self, SEL _cmd) { return (*(int *)((char *)self + OBJC_IVAR_$_Person$_age)); }
static void _I_Person_setAge_(Person * self, SEL _cmd, int age) { (*(int *)((char *)self + OBJC_IVAR_$_Person$_age)) = age; }
```
我们看到系统自动为我们生成了成员变量`_age`、还有方法`-(int)age;`、`-(void)setAge:(int)age;`

使用@property属性就相当于下面的代码

Person.h文件

```objc
@interface Person : NSObject{
    int _age;
}

-(int)age;
-(void)setAge:(int)age;
@end
```

Person.m文件

```
#import "Person.h"
#import <objc/runtime.h>

@implementation Person

-(void)setAge:(int)age{
    _age = age;
}

-(int)age{
    return _age;
}
@end
```

<!-- ************************************************ -->
## <a id="content2"></a>@synthesize

关键字@synthesize的作用就是通知编译器为相关属性自动生成，成员变量、set、get方法。     
`@synthesize age=_age,height=_height;`      
这是个是默认属性平时开发时不用我们自己书写。     

在开发时有时我们会遇见同时重载set、get方法会报错，只重载其中一个就没问题。

<img src="/images/underlying/msgsend4.png" alt="img">

猜测：当我们同时重载了set、get方法时，编译器会认为我们要手动实现set、get，就不会自动给我们生成这些方法，包括成员变量。所以就会报错：<span style="color:red">Use of undeclared identifier '_age';</span>

如何解决这个问题呢？    
一种方式是：自己手写一个成员变量_age;    
```objc
@interface Person : NSObject{
    int _age;
}
@property (nonatomic, assign) int age;
```
还有一种方式是：使用@synthesize明确告诉编译器要生成成员变量、set、get方法。当然set、get方法已经手动实现，那么它会生成成员变量。这样同时重载set、get就不会报错了。

```objc
@interface Person : NSObject
@property (nonatomic, assign) int age;
@end
```

```objc
#import "Person.h"

@implementation Person

@synthesize age=_age;

-(void)setAge:(int)age{
    _age = age;
}

-(int)age{
    return _age;
}
@end
```






<!-- ************************************************ -->
## <a id="content3"></a>@dynamic

@dynamic是告诉编译器不用自动生成成员变量、set、get方法，等到运行时再添加方法实现。    
`@dynamic age;`    

我们看下面的代码

Person.h文件
```
@interface Person : NSObject;
@property (nonatomic, assign) int age;
@end
```

Person.m文件
```objc
#import "Person.h"
@implementation Person
@dynamic age;

@en
```

通过指令`xcrun -sdk iphoneos clang -arch arm64 -rewrite-objc Person.m`     
生成Person.cpp文件    

```objc
struct Person_IMPL {
	struct NSObject_IMPL NSObject_IVARS;
};
```
我们看到编译器并没有给我们生成_age成员变量，也没有给我们生成set、get方法。

<img src="/images/underlying/msgsend5.png" alt="img">


我们可以通过运行时，在程序运行期间动态的添加这些成员变量、set、get方法。


----------
>  行者常至，为者常成！


