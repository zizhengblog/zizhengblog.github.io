---
layout: post
title: "nil,Nil,NULL,[NSNull null]"
date: 2018-12-06
tag: Object-C
--- 



- [参考文章：https://blog.csdn.net/wzzvictory/article/details/18413519](https://blog.csdn.net/wzzvictory/article/details/18413519)


## 目录
- [NULL](#content1)
- [nil](#content2)
- [Nil](#content3)
- [[NSNull null]](#content4)
- [总结](#content5)



首先要说明的是，nil、Nil、NULL三个关键字和NSNull类都是表示空，只是用处不一样，具体的区别如下：



<!-- ************************************************ -->
## <a id="content1"></a>NULL

**一、声明位置**

stddef.h文件

**二、定义**

```
#undef NULL
#ifdef __cplusplus
#  if !defined(__MINGW32__) && !defined(_MSC_VER)
#    define NULL __null
#  else
#    define NULL 0
#  endif
#else
#  define NULL ((void*)0)
#endif
```

其中__cplusplus表示 C++ 代码，对于iOS开发者来说，通常 NULL 的定义就是：

```
#  define NULL ((void*)0)
```

因此，NULL本质上是：(void*)0

**三、用处及含义**

NULL表示C指针为空

**四、示例**

`char *string = NULL;`



<!-- ************************************************ -->
## <a id="content2"></a>nil

**一、声明位置**

objc.h文件

**二、定义**

```
#ifndef nil
# if __has_feature(cxx_nullptr)
#   define nil nullptr
# else
#   define nil __DARWIN_NULL
# endif
#endif
```
其中`__has_feature(cxx_nullptr)`用于判断C++中是否有nullptr特性，对于iOS开发者来说，nil的定义形式为：`#   define nil __DARWIN_NULL`

就是说nil最终是`__DARWIN_NULL`的宏定义， `__DARWIN_NULL`是定义在_types.h中的宏，其定义形式如下：

```
#ifdef __cplusplus
#ifdef __GNUG__
#define __DARWIN_NULL __null
#else /* ! __GNUG__ */
#ifdef __LP64__
#define __DARWIN_NULL (0L)
#else /* !__LP64__ */
#define __DARWIN_NULL 0
#endif /* __LP64__ */
#endif /* __GNUG__ */
#else /* ! __cplusplus */
#define __DARWIN_NULL ((void *)0)
#endif /* __cplusplus */
```

非C++代码的 `__DARWIN_NULL`最终定义形式如下：
`#define __DARWIN_NULL ((void *)0)`

也就是说， nil本质上是：(void *)0


**三、用处及含义**

用于表示指向Objective-C中对象的指针为空

**四、示例**

```
NSString *string = nil;
id anyObject = nil;
```


<!-- ************************************************ -->
## <a id="content3"></a>Nil


**一、声明位置**

objc.h文件

**二、定义**

```
#ifndef Nil
# if __has_feature(cxx_nullptr)
#   define Nil nullptr
# else
#   define Nil __DARWIN_NULL
# endif
#endif
和上面讲到的nil一样， Nil本质上也是：(void *)0
```

**三、用处及含义**

用于表示Objective-C类（Class）类型的变量值为空

**四、示例**

```
Class anyClass = Nil;
```



<!-- ************************************************ -->
## <a id="content4"></a>[NSNull null]


**一、声明位置**

NSNull.h文件

**二、定义**

```
@interface NSNull : NSObject <NSCopying, NSSecureCoding>
 
+ (NSNull *)null;
 
@end
```

**三、用处及含义**

从定义中可以看出，NSNull是一个Objective-C类，只不过这个类相当特殊，因为它表示的是空值，即什么都不存。它也只有一个单例方法+[NSUll null]。该类通常用于在集合对象中保存一个空的占位对象。

**四、示例**

我们通常初始化NSArray对象的形式如下：

```
NSArray *arr = [NSArray arrayWithObjects:@"wang",@"zz",nil];
```

当NSArray里遇到nil时，就说明这个数组对象的元素截止了， 即NSArray只关注nil之前的对象，nil之后的对象会被抛弃。比如下面的写法：
```
NSArray *arr = [NSArray arrayWithObjects:@"wang",@"zz",nil,@"foogry"];
```

这是NSArray中只会保存wang和zz两个字符串，foogry字符串会被抛弃。

这种情况，就可以使用NSNull实现：

```
NSArray *arr = [NSArray arrayWithObjects:@"wang",@"zz",[NSNull null],@"foogry"];
```

<!-- ************************************************ -->
## <a id="content5"></a>总结

从前面的介绍可以看出，不管是NULL、nil还是Nil，它们本质上都是一样的，都是(void *)0，只是写法不同。这样做的意义是为了区分不同的数据类型，比如你一看到用到了NULL就知道这是个C指针，看到nil就知道这是个Objective-C对象，看到Nil就知道这是个Class类型的数据。





----------
>  行者常至，为者常成！



