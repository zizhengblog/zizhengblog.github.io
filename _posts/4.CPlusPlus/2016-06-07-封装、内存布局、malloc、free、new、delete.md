---
layout: post
title: "7、封装、内存布局、malloc、free、new、delete"
date: 2016-06-07
description: "封装、内存布局、malloc、free、new、delete"
tag: C++
---











## 目录

* [封装](#content1)
* [内存空间布局](#content2)
* [对象的内存](#content3)
* [memset的使用](#content4)
* [构造函数](#content5)
* [默认情况下，成员变量的初始化](#content6)
* [析构函数](#content7)





<!-- ************************************************ -->
## <a id="content1"></a>封装

成员变量私有化，提供公共的getter和setter给外界去访问成员变量     

有些变量我们不希望外界访问，那么我们就把该变量私有化，这样的好处是：     
外界无法直接访问    

可以过滤数据，比如下面的age赋值小于0时就直接返回     

```
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

**一、内存空间分布**

每个应用都有自己独立的内存空间，其内存空间一般都有以下几大区域

代码段(代码区)    
✓ 用于存放代码

数据段(全局区)    
✓ 用于存放全局变量等

堆空间     
✓ 需要主动去申请和释放

栈空间     
✓ 每调用一个函数就会给它分配一段连续的栈空间，等函数调用完毕后会自动回收这段栈空间     
✓ 自动分配和回收


**二、堆空间**

在程序运行过程，为了能够自由控制内存的生命周期、大小，会经常使用堆空间的内存       
堆空间的申请\释放       
malloc \ free       
new    \ delete       
new [] \ delete []       


注意       
申请堆空间成功后，会返回那一段内存空间的地址       
申请和释放必须是1对1的关系，不然可能会存在内存泄露       

现在的很多高级编程语言不需要开发人员去管理内存(比如Java)，屏蔽了很多内存细节，利弊同时存在       
利:提高开发效率，避免内存使用不当或泄露       
弊:不利于开发人员了解本质，永远停留在API调用和表层语法糖，对性能优化无从下手       

堆空间申请
```
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
```

堆空间的初始化
```
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
```


讲解案例
```
void heapInit2(void){
    //这种写法存在问题，申请了一个字节，但p认为自己指向了4个字节
    int * p = (int*)malloc(1);
    
    //赋值会覆盖掉其他3个字节的内容，导致不可预料的事情发生
    *p = 0xaabbccdd;
    
    //打印正常，是因为aabbccdd确实存在于内存中
    cout<<"*p = "<<*p<<endl;
    
    free(p);
}
```

```
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

对象的内存可以存在于3种地方     
全局区(数据段):全局变量     
栈空间:函数里面的局部变量     
堆空间:动态申请内存(malloc、new等)     


```
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

memset函数是将较大的数据结构(比如对象、数组等)内存清零的比较快的方法

```
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
    
    {
        int size = sizeof(int);
        int *p = (int *)malloc(size);
        // memory set
        // 从p开始的4个字节, 每个字节都存放0
        memset(p, 0, size);
        cout << *p << endl;
        free(p);
    }

    {
        /**
         memset函数是将较大的数据结构(比如对象、数组等)内存清零的比较快的方法
         */
        Person person;
        person.m_id = 1;
        person.m_age = 20;
        person.m_height = 180;
        memset(&person, 0, sizeof(person));
    }
    
    
    {
        /**
         memset函数是将较大的数据结构(比如对象、数组等)内存清零的比较快的方法
         */
        Test tests[3] = { {1,20,180} , {2,21,181} , {2,23,184} };
        memset(tests, 0, sizeof(tests));
    }

}
```

memcpy 的使用 

```
void memcpyUsing(){
    //memcpy指的是C和C++使用的内存拷贝函数，函数原型为
    //void *memcpy(void *destin, void *source, unsigned n)；
    //函数的功能是从源内存地址的起始位置开始拷贝若干个字节到目标内存地址中，
    //即从源source中拷贝n个字节到目标destin中。
        
    //    函数原型
    //    void *memcpy(void *destin, void *source, unsigned n);
    //    参数
    //    destin-- 指向用于存储复制内容的目标数组，类型强制转换为 void* 指针。
    //    source-- 指向要复制的数据源，类型强制转换为 void* 指针。
    //    n-- 要被复制的字节数
}
```



<!-- ************************************************ -->
## <a id="content5"></a>构造函数

构造函数(也叫构造器)，在对象创建的时候自动调用，一般用于完成对象的初始化工作

特点    
函数名与类同名，无返回值(void都不能写)，可以有参数，可以重载，可以有多个构造函数      
一旦自定义了构造函数，必须用其中一个自定义的构造函数来初始化对象       

注意    
通过malloc分配的对象不会调用构造函数    

一个广为流传的、很多教程\书籍都推崇的错误结论:    
错误理解：默认情况下，编译器会为每一个类生成空的无参的构造函数    
正确理解:在某些特定的情况下，编译器才会为类生成空的无参的构造函数    
(哪些特定的情况?以后再提)    

```
class Person{
public:
    int age;
    int height;
    
    
    Person(){
        cout<<"Person()"<<endl;
    }
    
    Person(int age){
        cout<<"Person(int age)"<<endl;
        this->age = age;
    }
    
    Person(int age,int height){
        cout<<"Person(int age,int height)"<<endl;
        this->age = age;
        this->height = height;
    }
    
    void display(){
        cout<<"age is "<<age<<endl;
        cout<<"height is "<<height<<endl;
    }
};

```

```
void personConstructor(){
    Person person1;//Person person1 = Person();
    person1.display();
    
    Person person2(10);//Person person2 = Person(10);
    person2.display();
    
    Person person3(20,180);//Person person3 = Person(2,180);
    person3.display();
}
```


```
// 全局区
//Person g_person1; // Person()
//Person g_person2(); // 这是一个函数声明，函数名叫g_person2，无参，返回值类型是Person
//Person g_person3(10); // Person(int age)
void personConstructor2(){
    // 栈空间
    Person person1; // Person()
    Person person2(); // 函数声明，函数名叫person2，无参，返回值类型是Person
    Person person3(20);  // Person(int age)

    // 堆空间
    Person *p1 = new Person; // Person()
    Person *p2 = new Person(); // Person()
    Person *p3 = new Person(30);  // Person(int age)
}
```

<!-- ************************************************ -->
## <a id="content6"></a>默认情况下，成员变量的初始化

```
class Person1{
public:
    int age;
};



// 全局区（成员变量初始化为0）
Person1 g_person11;

void memberInit1(){
    //栈空间
    Person1   p11;                      //成员变量不会被初始化
    Person1   p11_2 = Person1();        //成员变量初始化为0

    Person1 * p12 = new Person1;        //成员变量不会被初始化
    Person1 * p13 = new Person1();      //成员变量初始化为0

    Person1 * p14 = new Person1[3];     //成员变量不会被初始化
    Person1 * p15 = new Person1[3]();   //3个Person1对象的成员变量都初始化为0
    Person1 * p16 = new Person1[3]{};   //3个Person1对象的成员变量都初始化为0
}
```

如果自定义了构造函数，除了全局区，其他内存空间的成员变量默认都不会被初始化，需要开发人员手动初始化

```
class Person2{
public:
    int age;

    //构造函数
    Person2(){
        //可以通过这种方式将所有的成员变量的内存赋值为0
        memset(this, 0, sizeof(Person2));
    }

};

void memberInit2(){
    Person2 person2_1;
    cout<<"person2_1.age is "<<person2_1.age<<endl;

    Person2 * person2_2 = new Person2;
    cout<<"person2_2->age is "<<person2_2->age<<endl;

}
```


<!-- ************************************************ -->
## <a id="content7"></a>析构函数

析构函数(也叫析构器)，在对象销毁的时候自动调用，一般用于完成对象的清理工作      
 
特点      
函数名以~开头，与类同名，无返回值(void都不能写)，无参，不可以重载，有且只有一个析构函数      
 
注意      
通过malloc分配的对象free的时候不会调用构造函数      
构造函数、析构函数要声明为public，才能被外界正常使用      

```
class Person3{
public:
    int age;
    
    //对象创建完毕的时候调用
    Person3(){
        cout<<"Person3()"<<endl;
    }
    
    //重载构造函数
    Person3(int age){
        cout<<"Person3(int age)"<<endl;
    }
    
    //析构函数对象销毁(内存被回收)的时候调用
    ~Person3(){
        cout<<"~Person3()"<<endl;
    }
    
    //析构函数不存在重载函数
//    ~Person3(int age){
//        cout<<"~Person3(int age)"<<endl;
//    }
};

void deconstructor(){
    
    Person3 person;
    
    Person3 * p3 = new Person3();
    
    delete p3;
}

```


----------
>  行者常至，为者常成！


