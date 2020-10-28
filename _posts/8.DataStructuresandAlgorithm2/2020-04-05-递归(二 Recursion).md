---
layout: post
title: "14、递归(二 Recursion)"
date: 2020-04-03
description: ""
tag: 数据结构与算法（二）
---






## 目录

* [上楼梯](#content1)
* [汉诺塔（Hanoi）](#content2)
* [递归转非递归](#content3)
* [尾调用](#content4)





<!-- ************************************************ -->
## <a id="content1"></a>上楼梯

楼梯有 n 阶台阶，上楼可以一步上 1 阶，也可以一步上 2 阶，走完 n 阶台阶共有多少种不同的走法？

假设 n 阶台阶有 f(n) 种走法，第 1 步有 2 种走法   
✓ 如果上 1 阶，那就还剩 n – 1 阶，共 f(n – 1) 种走法   
✓ 如果上 2 阶，那就还剩 n – 2 阶，共 f(n – 2) 种走法    

所以 f(n) = f(n – 1) + f(n – 2)

<img src="/images/DataStructurs2/recursion8.png" alt="img">

```
/// 上n阶楼梯有多少种走法
/// @param n n阶
-(int)climbStairs:(int)n{
    if (n<=2) return n;
    return [self climbStairs:n-1] + [self climbStairs:n-2];
}
```

跟斐波那契数列几乎一样，因此优化思路也是一致的

```
/// 上n阶楼梯有多少种走法
/// @param n n阶
-(int)climbStairs2:(int)n{
    if (n<=2) return n;
    
    int first  = 1;
    int second = 2;
    
    for (int i = 3; i<=n; i++) {
        second = second + first;
        first  = second - first;
    }
    
    return second;
}
```
 
<!-- ************************************************ -->
## <a id="content2"></a>汉诺塔（Hanoi） 

**一、问题描述**

编程实现把 A 的 n 个盘子移动到 C（盘子编号是 [1, n] ）    
每次只能移动1个盘子   
大盘子只能放在小盘子下面     

<img src="/images/DataStructurs2/recursion9.png" alt="img">

**二、示例演示**

<center style="font-weight:bold">1个盘子</center>
<img src="/images/DataStructurs2/recursion10.png" alt="img">

<center style="font-weight:bold">2个盘子</center>
<img src="/images/DataStructurs2/recursion11.png" alt="img">

<center style="font-weight:bold">3个盘子</center>
<img src="/images/DataStructurs2/recursion12.png" alt="img">
<img src="/images/DataStructurs2/recursion13.png" alt="img">


**三、思路及实现**


 

<!-- ************************************************ -->
## <a id="content3"></a>递归转非递归

<!-- ************************************************ -->
## <a id="content4"></a>尾调用


----------
>  行者常至，为者常成！


