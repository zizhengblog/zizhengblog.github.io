---
layout: post
title: "Runtime（二）：Class详解"
date: 2018-07-04
description: "Runtime（二）：Class详解"
tag: Objective-C
---







## 目录

- [Class的数据结构](#content1)   
- [Class的数据查看](#content2)   



<!-- ************************************************ -->
## <a id="content1"></a>Class的数据结构

**一、Class的结构**

<img src="/images/underlying/oc16.png" alt="img">

**二、class_rw_t**

class_rw_t里面的 methods、properties、protocols是二维数组，是可读可写的，包含了类的初始内容，分类的内容。

<img src="/images/underlying/oc17.png" alt="img">

**三、class_ro_t**

class_ro_t里面的baseMethodList、baseProtocols、ivars、baseProperties是一维数组，是只读的包含了类的初始内容。

<img src="/images/underlying/oc18.png" alt="img">

**四、method_t**

method_t 是对方法/函数的封装

<img src="/images/underlying/oc19.png" alt="img">

1、SEL 选择器代表方法/函数名，底层结构与char*类似     
```
typedef struct objc_selector *SEL;
```

```objc
//获取SEL变量
SEL sel = @selector(btnClick:);
SEL sel1 = sel_registerName("btnClick:");

//SEL变量的使用
[btn addTarget:self action:sel1 forControlEvents:UIControlEventTouchUpInside];

//从SEL变量获取字符串
NSString * name =  NSStringFromSelector(sel);
const char * name1 = sel_getName(sel);
```
不同类中相同名字的方法，所对应的方法选择器是相同的
```objc
//0x1c134ab3f,0x1c134ab3f,0x1c134ab3f
NSLog(@"%p,%p,%p",@selector(test),@selector(test), sel_registerName("test"));
```

2、types包含了函数返回值、参数编码的字符串  

|返回值|参数1|参数2|...|参数n|   

v16@0:8    

|v|返回值为void|
|16|代表返回值和参数共占16字节|
|@|参数1为id类型|
|0|参数1占用内存起始位置为0|
|:|参数2为SEL类型|
|8|参数2占用内存起始位置为8|




ios中提供了一个叫@encode的指令，可以将具体的类型表示成字符串编码
```objc
//  :
NSLog(@"%s",@encode(SEL));
```

|类型|encode|
|SEL|:|
|int|i|
|void|v|
|id|@|
|BOOL|B|
|Class|#|


3、IMP 指向函数的指针     
看下objc源码里的定义     
```c
// id functionName(id,SEL,...) 把指向这种类型的函数指针定义为IMP
typedef id _Nullable (*IMP)(id _Nonnull, SEL _Nonnull, ...); 
```

4、示例图

<img src="/images/underlying/oc20.png" alt="img">

<img src="/images/underlying/oc21.png" alt="img">

**五、cache_t**     

Class内部结构中有个方法缓存cache_t,用<span style="color:red">散列表</span>来缓存曾经调用过的方法，可以提高方法的查找速度  

<img src="/images/underlying/oc22.png" alt="img">

散列表的结构

<img src="/images/underlying/oc23.png" alt="img">





<!-- ************************************************ -->
## <a id="content2"></a>Class的数据查看


- [参考文章：获取Class数据结构信息](https://jianghuhike.github.io/18710.html)




----------
>  行者常至，为者常成！


