---
layout: post
title: "7、封装、内存布局、malloc、free、new、delete"
date: 2019-11-13
description: "封装、内存布局、malloc、free、new、delete"
tag: C++
---


<h6>版权声明：本文为博主原创文章，未经博主允许不得转载。</h6>








## 目录

* [封装](#content1)
* [内存空间布局](#content2)
* [对象的内存](#content3)
* [memset的使用](#content4)






<!-- ************************************************ -->
## <a id="content1"></a>封装
```
/**
 成员变量私有化，提供公共的getter和setter给外界去访问成员变量
 
 有些变量我们不希望外界访问，那么我们就把该变量私有化，这样的好处是：
 外界无法直接访问
 可以过滤数据，比如下面的age赋值小于0时就直接返回
 */

class Person{
private:
    int _age;
public:
    int m_id;
    int m_age;
    int m_height;
    
    void setAge(int age){
        if (age<0) {
            cout<<"age can not be low 0"<<endl;
        }
        _age = age;
    }
    
    int getAge(){
        return _age;
    }
};


void capsulation(){
    Person person;
    
    person.setAge(10);
    
    int age = person.getAge();
    
    cout<<"age is "<<age<<endl;
}

```


<!-- ************************************************ -->
## <a id="content2"></a>内存空间布局
```
 /**
  每个应用都有自己独立的内存空间，其内存空间一般都有以下几大区域
  
  
  代码段(代码区)
  ✓ 用于存放代码
  
  
  数据段(全局区)
  ✓ 用于存放全局变量等
  
  
  栈空间
  ✓ 每调用一个函数就会给它分配一段连续的栈空间，等函数调用完毕后会自动回收这段栈空间
  ✓ 自动分配和回收
  
  
  堆空间
  ✓ 需要主动去申请和释放
  */



/**
 堆空间
 
 在程序运行过程，为了能够自由控制内存的生命周期、大小，会经常使用堆空间的内存
 
 ◼ 堆空间的申请\释放
 malloc \ free
 new \ delete
 new [] \ delete []
 
 
 ◼注意
 申请堆空间成功后，会返回那一段内存空间的地址
 申请和释放必须是1对1的关系，不然可能会存在内存泄露
 ◼ 现在的很多高级编程语言不需要开发人员去管理内存(比如Java)，屏蔽了很多内存细节，利弊同时存在
 利:提高开发效率，避免内存使用不当或泄露
 弊:不利于开发人员了解本质，永远停留在API调用和表层语法糖，对性能优化无从下手
 */

//堆空间申请
void heapApply(){
    //申请4字节的堆空间内存
    int *p = (int*)malloc(4);
    //将申请的4个字节初始化为1
    //0x00 00 00 01
    *p = 1;
    cout<<"*p = "<<*p<<endl;
    
    //申请时分配了多少空间，就释放多少空间
    free(p);
    
    
    //申请4字节的堆空间内存
    char *p2 = (char*)malloc(4);
    
    //将申请的第一个字节初始化为1 0x01
    *p2 = 1;
    
    //将申请的第二个字节初始化为2 0x02
    *(p2+1) = 2;
    
    //将申请的第三个字节初始化为3 0x03
    *(p2+2) = 3;

    //将申请的第四个字节初始化为4 0x04
    *(p2+3) = 4;
    
    cout<<"*p2"<<*p2<<endl;

    //申请时分配了多少空间，就释放多少空间
    free(p2);
}



//堆空间的初始化
void heapInit(){
    //向堆空间申请4字节大小的内存
    /**
     指针变量p1位于栈空间占用8字节（64位系统）
     p1内存储的值，指向堆空间，该堆空间的大小为4字节
     */
    //*p1 未初始化 根据平台的不同会不同，有的平台会初始化。兼容所有不推荐这么写，代码移植到其它平台会出现问题。
    int *p1 = (int*)malloc(sizeof(int));
    cout<<"*p1 = "<<*p1<<endl;
    free(p1);
    
    
    //将*p2的每一个字节都初始化为0
    int *p2 = (int*)malloc(sizeof(int));
    memset(p2, 0, sizeof(int));
    cout<<"*p2 = "<<*p1<<endl;
    free(p2);
}


//讲解案例
void heapInit2(void){
    //这种写法存在问题，申请了一个字节，但p认为自己指向了4个字节
    int * p = (int*)malloc(1);
    
    //赋值会覆盖掉其他3个字节的内容，导致不可预料的事情发生
    *p = 0xaabbccdd;
    
    //打印正常，是因为aabbccdd确实存在于内存中
    cout<<"*p = "<<*p<<endl;
    
    free(p);
}



void heapInit3(){
    // 没有初始化
    int *p1 = new int;//不同的平台会有差异
    
    // 初始化为0
    int *p2 = new int();//推荐写法
    
    // 初始化为5
    int *p3 = new int(5);
    
    // 没有初始化
    int *p4 = new int[3];
    
    // 全部元素初始化为0
    int *p5 = new int[3]();
    
    // 全部元素初始化为0
    int *p6 = new int[3]{};
    
    // 首元素初始化为5，其他元素初始化为0
    int *p7 = new int[3]{ 5 };
    
    cout << p7[0] << endl;
    cout << p7[1] << endl;
    cout << p7[2] << endl;
}

```


<!-- ************************************************ -->
## <a id="content3"></a>对象的内存
```
/**
 ◼ 对象的内存可以存在于3种地方
 全局区(数据段):全局变量
 栈空间:函数里面的局部变量
 堆空间:动态申请内存(malloc、new等)
 */

// 全局区
Person g_person;

void classMemory(){
    // 栈空间
    Person person;
    
    // 堆空间
    Person *p = new Person();
    p->m_height = 180;
    
    delete p;
}
```


<!-- ************************************************ -->
## <a id="content4"></a>memset的使用
```
// memset函数是将较大的数据结构(比如对象、数组等)内存清零的比较快的方法


struct Test{
    int m_id;
    int m_age;
    int m_height;
    
    void setAge(int age){
        if (age<0) {
            cout<<"age can not be low 0"<<endl;
        }
        m_age = age;
    }
    
    int getAge(){
        return m_age;
    }
};


void memsetUsing(){
    int size = sizeof(int);
    int *p = (int *)malloc(size);
    // memory set
    // 从p开始的4个字节, 每个字节都存放0
    memset(p, 0, size);
    cout << *p << endl;
    free(p);
    
    
    //memset函数是将较大的数据结构(比如对象、数组等)内存清零的比较快的方法
    Person person;
    person.m_id = 1;
    person.m_age = 20;
    person.m_height = 180;
    memset(&person, 0, sizeof(person));
    
    
    
    //memset函数是将较大的数据结构(比如对象、数组等)内存清零的比较快的方法
    Test tests[3] = {
        {1,20,180},
        {2,21,181},
        {2,23,184}
    };
    memset(tests, 0, sizeof(tests));
}

```




----------
>  行者常至，为者常成！


