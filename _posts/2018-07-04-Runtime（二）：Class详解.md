---
layout: post
title: "Runtime（二）：Class详解"
date: 2018-07-04
description: "Runtime（二）：Class详解"
tag: 底层原理
---


<h6>
  版权声明：本文为博主原创文章，未经博主允许不得转载。
  <a target="_blank" href="https://jianghuhike.github.io/1874.html">
  原文地址：https://jianghuhike.github.io/1874.html 
  </a>
</h6>




## 目录

- [Class的数据结构](#content1)   
- [Class的数据查看](#content2)   



<!-- ************************************************ -->
## <a id="content1"></a>Class的数据结构

一、Class的结构

<img src="/images/underlying/oc16.png" alt="img">

二、class_rw_t 

class_rw_t里面的 methods、properties、protocols是二维数组，是可读可写的，包含了类的初始内容，分类的内容。

<img src="/images/underlying/oc17.png" alt="img">

三、class_ro_t

class_ro_t里面的baseMethodList、baseProtocols、ivars、baseProperties是一维数组，是只读的包含了类的初始内容。





<!-- ************************************************ -->
## <a id="content2"></a>Class的数据查看






----------
>  行者常至，为者常成！


