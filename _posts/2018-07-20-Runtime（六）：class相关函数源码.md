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


<!-- ************************************************ -->
## <a id="content1"></a>class函数

先看下源码实现

```objc
+ (Class)class {
    return self;
}

- (Class)class {
    return object_getClass(self);
}
```

可以看出类对象调用时直接返回self，即将自身返回也就是类对象。      
实例对象调用时通过object_getClass函数返回的也是类对象。     
所以不管类对象还是实例对象调用class方法返回的都是类对象。      

```
NSObject * obj = [[NSObject alloc] init];

//不管调用多少次class返回的都是类对象
[[[obj class] class] class];

[obj class];//返回类对象，之后调用class都返回自身
```















----------
>  行者常至，为者常成！


