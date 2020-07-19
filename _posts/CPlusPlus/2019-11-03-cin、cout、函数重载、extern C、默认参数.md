---
layout: post
title: "2、cin、cout、函数重载、extern C、默认参数"
date: 2019-11-03
description: "cin、cout、函数重载、extern C、默认参数"
tag: C++
---


<h6>版权声明：本文为博主原创文章，未经博主允许不得转载。</h6>








## 目录

* [C++中常使用cin、cout进行控制台的输入、输出](#content1)
* [函数重载](#content2)
* [extern "c"](#content3)
* [默认参数](#content4)






<!-- ************************************************ -->
## <a id="content1"></a>C++中常使用cin、cout进行控制台的输入、输出

```
void cout_cin_func(){
    //cin用的右移运算符>>，cout用的是左移运算符<<
    //endl是换行的意思
    
    
    int age = 0b1100;
    
    cout<<"age is "<<age<<endl;
    
    //
    int b = age >>2;
    
    cout<<"b is "<< b << endl;
    
    // 等待键盘输入
    //    getchar();
    
}
```

<!-- ************************************************ -->
## <a id="content2"></a>函数重载

 规则

 函数名相同

 参数个数不同、参数类型不同、参数顺序不同
 
 注意

 返回值类型与函数重载无关，调用函数时，实参的隐式类型转换可能会产生二义性
 
 本质

 采用了name mangling或者叫name decoration技术，C++编译器默认会对符号名(变量名、函数名等)进行改编、修饰，有些地方翻译为“命名倾轧”，重载时会生成多个不同的函数名，不同编译器(MSVC、g++)有不同的生成规则，通过IDA打开可以看到

```
//overLoadFunc_i
void overLoadFunc(int a){
    cout<<"overLoadFunc_int"<<endl;
    cout<<"a is"<<a<<endl;
}


//overLoadFunc_i_i
void overLoadFunc(int a,int b){
    
    cout<<"overLoadFunc_int_int"<<endl;
    
    cout<<"a is"<<a<<"b is"<<b<<endl;
}
```

<!-- ************************************************ -->
## <a id="content3"></a>extern "c"

```
/**
 被extern "C"修饰的代码会按照C语言的方式去编译
 ◼如果函数同时有声明和实现，要让函数声明被extern "C"修饰，函数实现可以不修饰
 
 由于C、C++编译规则的不同，在C、C++混合开发时，可能会经常出现以下操作
 C++在调用C语言API时，需要使用extern "C"修饰C语言的函数声明
 
 有时也会在编写C语言代码中直接使用extern “C” ，这样就可以直接被C++调用
 */

//如果直接这么声明
//编译时会报Undefined symbol: twoDayCFileFunc(int, int)
//因为C++的name mangling技术，twoDayCFileFunc 方法可能是twoDayCFileFunc_int_int
//void twoDayCFileFunc(int,int);

//方式一：
//extern "C" void twoDayCFileFunc(int,int);

//方式二：
//extern "C" {
//    void twoDayCFileFunc(int,int);
//}

//extern "C" {
// #include "twoDayCFile.h"
//}

/**
 方式三：
 通过使用宏:__cplusplus来区分C/C++环境
 每个C++文件，在文件的开始都默认添加了宏 __cplusplus
 所以要想在编写通用的C库，既能在C环境调用，又能在C++环境调用，那么C的头文件应该这样设计
 
 
 #ifndef twoDayCFile_h
 #define twoDayCFile_h
 
 #ifdef __cplusplus
 extern "C"{
 #endif

 #include <stdio.h>
 void twoDayCFileFunc(int a, int b);

 #ifdef __cplusplus
 }
 #endif
 
 #endif
 */

/**
#pragma once

我们经常使用#ifndef、#define、#endif来防止头文件的内容被重复包含

#pragma once可以防止整个文件的内容被重复包含

区别
#ifndef、#define、#endif受C\C++标准的支持，不受编译器的任何限制
有些编译器不支持#pragma once(较老编译器不支持，如GCC 3.4版本之前)，兼容性不够好 #ifndef、#define、#endif可以针对一个文件中的部分代码，而#pragma once只能针对整个文件
*/
```

twoDayCFile.h
```
#ifndef twoDayCFile_h
#define twoDayCFile_h

#ifdef __cplusplus
extern "C"{
#endif

#include <stdio.h>
void twoDayCFileFunc(int a, int b);

#ifdef __cplusplus
}
#endif

#endif
```

twoDayCFile.c
```
#include "twoDayCFile.h"
#include <stdio.h>

void twoDayCFileFunc(int a, int b){
    
    printf("a + b = %d\n",a+b);
    
}
```

twoDay.cpp中调用
```
#include "twoDayCFile.h"

void externCFunc(void){
    twoDayCFileFunc(10,20);
}
```


<!-- ************************************************ -->
## <a id="content4"></a>默认参数

```
/**
 C++允许函数设置默认参数，在调用时可以根据情况省略实参。规则如下:
 默认参数只能按照右到左的顺序
 如果函数同时有声明、实现，默认参数只能放在函数声明中
 默认参数的值可以是常量、全局符号(全局变量、函数名)
 
 
 函数重载、默认参数可能会产生冲突、二义性(建议优先选择使用默认参数)
 */

void defaultParaFunc(int a,int b){
    
    cout<<"a + b = " << a + b <<endl;
}


void defaultParaFunc(int a,int b, int c = 30){
    
    cout << "a + b + c = "<<a + b + c<<endl;
}


void defaultParaTest(){
    cout<<"defaultParaTest"<<endl;
}

void defaultParaFunc(void(*p)() = defaultParaTest){
    p();
}

```







----------
>  行者常至，为者常成！


