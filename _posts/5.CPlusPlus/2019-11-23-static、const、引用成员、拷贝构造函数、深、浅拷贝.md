---
layout: post
title: "12、static、const、引用成员、拷贝构造函数、深、浅拷贝"
date: 2019-11-23
description: "static、const、引用成员、拷贝构造函数、深、浅拷贝"
tag: C++
---


<h6>版权声明：本文为博主原创文章，未经博主允许不得转载。</h6>








## 目录

* [静态成员](#content1)
* [静态成员经典应用 - 单例模式](#content2)
* [const成员](#content3)
* [引用类型成员](#content4)
* [拷贝构造函数](#content5)
* [深拷贝、浅拷贝](#content6)



<!-- ************************************************ -->
## <a id="content1"></a>静态成员

```
/**
 ◼ 静态成员:被static修饰的成员变量\函数
 可以通过对象(对象.静态成员)、对象指针(对象指针->静态成员)、类访问(类名::静态成员)
 
 ◼ 静态成员变量
 存储在数据段(全局区，类似于全局变量)，整个程序运行过程中只有一份内存
 对比全局变量，它可以设定访问权限(public、protected、private)，达到局部共享的目的
 必须初始化，必须在类外面初始化，初始化时不能带static，如果类的声明和实现分离(在实现.cpp中初始化)
 
 ◼ 静态成员函数
 内部不能使用this指针(this指针只能用在非静态成员函数内部)
 不能是虚函数(虚函数只能是非静态成员函数)
 内部不能访问非静态成员变量\函数，只能访问静态成员变量\函数
 非静态成员函数内部可以访问静态成员变量\函数
 构造函数、析构函数不能是静态
 当声明和实现分离时,实现部分不能带static
 */

/**
 静态成员变量和静态函数可以通过类名，通过类名访问时有可能对象还未创建，所以所有与对象相关的都不能使用static修饰
 */

class Car{
private:
    int m_price;
    static int ms_count;
public:
    static int getCount(){
        return ms_count;
    }
    
    Car(int price = 0):m_price(price){
        //此处应该考虑线程问题
        ms_count++;
    }
    
    ~Car(){
        //此处应该考虑线程问题
        ms_count--;
    }
};

//初始化静态成员变量
int Car::ms_count = 0;


void staticMemberUsing(){
    Car car1;
    //car1.ms_count;//访问方式一
    
    Car * p = new Car();
    //p->ms_count;//访问方式二
    
    //Car::ms_count;//访问方式三
    
//    cout<<"Car::ms_count is "<<Car::ms_count<<endl;
    
    cout<<"Car::getCount() is "<<Car::getCount()<<endl;
    
    delete p;
    
    cout<<"Car::getCount() is "<<Car::getCount()<<endl;
}

```


<!-- ************************************************ -->
## <a id="content2"></a>静态成员经典应用 - 单例模式

```
/*
单例模式：
在程序运行过程中，可能会希望某些类的实例对象永远只有一个

1.把构造函数私有化
2.定义一个私有的静态成员变量指针，用于指向单例对象
3.提供一个公共的返回单例对象的静态成员函数
*/

// C++：静态成员函数
// Java、OC：类方法

class Rocket{
private:
    //构造函数和析构函数私有化禁止外部调用
    Rocket(){}
    ~Rocket(){}
    static Rocket * r;
public:
    static Rocket* shareRocket(){
        //考虑线程安全问题
        if (r == nullptr) {
            r = new Rocket();
        }
        return r;
    }
    
    static void deleteRocket(){
        //考虑线程安全问题
        if(r != nullptr){
            delete r;
            r = nullptr;
        }
    }
};

//初始化静态成员变量
Rocket* Rocket::r = nullptr;

void shareInstanceUsing(){
    Rocket * rocket0 = Rocket::shareRocket();
    Rocket * rocket1 = Rocket::shareRocket();
    Rocket * rocket2 = Rocket::shareRocket();
    Rocket * rocket3 = Rocket::shareRocket();

    cout<<"rocket0 is "<<rocket0<<endl;
    cout<<"rocket1 is "<<rocket1<<endl;
    cout<<"rocket2 is "<<rocket2<<endl;
    cout<<"rocket3 is "<<rocket3<<endl;
    
    Rocket::deleteRocket();
    
    Rocket * rocket4 = Rocket::shareRocket();
    Rocket * rocket5 = Rocket::shareRocket();


    cout<<"rocket4 is "<<rocket4<<endl;
    cout<<"rocket5 is "<<rocket5<<endl;


    

}
```



<!-- ************************************************ -->
## <a id="content3"></a>const成员

```

/**
 ◼ const成员:被const修饰的成员变量、非静态成员函数
 
 ◼ const成员变量
 必须初始化(类内部初始化)，可以在声明的时候直接初始化赋值
 非static的const成员变量还可以在初始化列表中初始化
 
 ◼ const成员函数(非静态)
 const关键字写在参数列表后面，函数的声明和实现都必须带const
 内部不能修改非static成员变量
 内部只能调用const成员函数、static成员函数
 非const成员函数可以调用const成员函数
 const成员函数和非const成员函数构成重载
 非const对象(指针)优先调用非const成员函数
 const对象(指针)只能调用const成员函数、static成员函数
 */

class Car{
public:
    int m_price;
    
    static int ms_size;
    
    //const 成员变量必须初始化,在类内部进行初始化
    const int mc_speed = 0;
    
    static const int msc_peple;
    
    
    Car():m_price(0){
        
    }
    
    //test 与 const修饰的test函数 构成重载
    void test(){
        cout<<"test()"<<endl;
    }
    
    void test() const{
        cout<<"test()const"<<endl;
    }
    
    static void testS(){
        
    }
    
    
    
    
    void test1(){
        //所有成员变量都可以访问
        this->m_price = 1;
        this->ms_size = 2;
        this->mc_speed;// = 3;
        this->msc_peple;// = 4;
        
        
        
        //非静态、非const修饰的函数可以调用 所有的成员函数
        //包括静态成员函数和const修饰的成员函数
        test();//优先调用非const修饰的test函数
        testS();
    }
    
    void test1()const{
        //不可以访问
//        this->m_price = 1;
        
        this->ms_size = 2;
        this->mc_speed;
        this->msc_peple;
        
        
        test();//优先调用const修饰的函数
        testS();
    }
    
    static void testS1(){
        
        //不可访问
//        m_price = 1;
        
        //可以访问
        ms_size = 10;
        
        //不可访问
//        mc_speed;
        
        //可以访问
        msc_peple;
        
        //静态成员函数，只能调用静态成员函数
        testS();
    }
    
    //const 与 static 不能同时作用域成员函数
//    static void testS2()const{
//
//    }

};

//静态成员变量必须在类外初始化
int Car::ms_size = 100;
const int Car::msc_peple = 5;


void constUsing(){
    Car car;
    car.test();//优先调用非const修饰的test函数
    
    const Car car2;
    car2.test();//只能调用const修饰的test函数

}

```

<!-- ************************************************ -->
## <a id="content4"></a>引用类型成员

```
/**
 ◼ 引用类型成员变量必须初始化(不考虑static情况)
 在声明的时候直接初始化
 通过初始化列表初始化
 */

class Person{
    int m_age;
    int & m_height = m_age;
    
    Person(int &height):m_height(height){
        
    }
};

```



<!-- ************************************************ -->
## <a id="content5"></a>拷贝构造函数

```
/**
 ◼ 拷贝构造函数是构造函数的一种
 ◼ 当利用已存在的对象创建一个新对象时(类似于拷贝)，就会调用新对象的拷贝构造函数进行初始化
 ◼ 拷贝构造函数的格式是固定的，接收一个const引用作为参数
 */

class Person3{
public:
    int m_age;
    int m_height;
    Person3(){
        cout<<"Person3()"<<endl;
    }

    //与构造函数构成重载
    //（如果不实现拷贝构造函数，也会默认进行拷贝）,系统的拷贝都是浅拷贝
    Person3(const Person3 &person):m_age(person.m_age),m_height(person.m_height){
         cout<<"Person3(const Person3 &person)"<<endl;
    }
};


class Student3:public Person3{
public:
    int m_score;
    
    Student3(){
        cout<<"Student3()"<<endl;
    }
    
    //Person3(student) 调用父类的拷贝构造函数
    //Person3(student) 引用的本质是指针，这里是父类指针指向子类对象
    //如果不实现拷贝构造函数，也会默认调用父类的拷贝构造函数
    Student3(const Student3 & student):Person3(student),m_score(student.m_score){
         cout<<"Student3(const Student3 & student)"<<endl;
    }
};




void copyConstructorUsing(){
    
    
    //调用构造函数(下面两种写法等价)
//    Person3 p1_1;
    Person3 p1_1 = Person3();
    
    //调用拷贝构造函数（利用已有对象初始化对象）(下面三种写法等价)
//    Person3 p1_2(p1_1);
//    Person3 p1_2 = Person3(p1_1);
    Person3 p1_2 = p1_1;
    
    //调用构造函数
    Person3 * p1_3 = new Person3();
    
    //调用拷贝构造函数（利用已有对象初始化对象）
    Person3 * p1_4 = new Person3(p1_1);

    
    cout<<"-----------1-----------"<<endl;

    
   
    
    //调用构造函数
    Person3 person2_1;
    Person3 person2_2;
    
    //因为person2_1与person2_2都已经创建完成，这里是赋值操作
    person2_1 = person2_2;
    
     cout<<"-----------2-----------"<<endl;

    
    //调用构造函数
    Student3 stu;
    
    //以下三种写法等价，调用拷贝构造函数
    Student3 stu1 = stu;
    Student3 stu2(stu);
    Student3 stu3 = Student3(stu);
    
    
    cout<<"-----------3-----------"<<endl;
    
    //调用构造函数
    Person3 person4_1;
    
    //没有产生新的对象，所以不会调用构造函数
    Person3 &person4_2 = person4_1;
    
    cout<<"-----------4-----------"<<endl;

}
```

<!-- ************************************************ -->
## <a id="content6"></a>深拷贝、浅拷贝

```
/**
 ◼编译器默认的提供的拷贝是浅拷贝(shallow copy)
 将一个对象中所有成员变量的值拷贝到另一个对象
 如果某个成员变量是个指针，只会拷贝指针中存储的地址值，并不会拷贝指针指向的内存空间
 可能会导致堆空间多次free的问题
 
 ◼如果需要实现深拷贝(deep copy)，就需要自定义拷贝构造函数
 将指针类型的成员变量所指向的内存空间，拷贝到新的内存空间
 */

class Truck{
private:
    int m_price;
    char * m_name;
    void copyName(const char* name){
        if (name == nullptr) {
            return;
        }
        
        m_name = new char[strlen(name)+1];
        strcpy(m_name, name);
    }
    
public:
    Truck(int price = 0,const char * name = nullptr):m_price(price){
//        if (name == nullptr) {
//            return;
//        }
//        m_name = new char [strlen(name)+1];
//        strcpy(m_name, name);
        
        copyName(name);
    }
    
    //自定义拷贝构造函数的意义是：完成深拷贝
    Truck(const Truck &truck){
        //为什么要用引用
        //如果不是引用，const Truck truck = otherTruck;//调用拷贝构造函数，发生递归调用
        
        //为什么用const修饰
        //const修饰参数的目的一般是保护变量，只允许内部使用，不允许内部修改
        
//        if (truck.m_name == nullptr) {
//            return;
//        }
//
//        m_name = new char[strlen(truck.m_name)+1];
//        strcpy(m_name, truck.m_name);
        
        copyName(truck.m_name);
        
    }
    
    ~Truck(){
        if (m_name !=nullptr) {
            delete [] m_name;
            m_name = nullptr;
        }
    }
};

void shallowAndDeepCopyUsing(){
    
    const char * name = "abc";
//    (lldb) po &name
//    0x00007ffeefbff598
//
//    (lldb) memory read/4xg 0x00007ffeefbff598
//    0x7ffeefbff598: 0x000000010001f7a8 0x00007ffeefbff5b0
//    0x7ffeefbff5a8: 0x0000000100001f29 0x00007ffeefbff5c0
//    (lldb) memory read/4xg 0x000000010001f7a8
//    0x10001f7a8: 0x7473657400636261 0x2874736574002928
//    0x10001f7b8: 0x500074736e6f6329 0x2928336e6f737265

    //此处的abc存放的地址是 0x000000010001f7a8
    
    Truck truck(100,name);
    
//    (lldb) po &truck
//    0x00007ffeefbff588
//
//    (lldb) memory read/4xg 0x00007ffeefbff588
//    0x7ffeefbff588: 0x00007ffe00000064 0x0000000100509d10
//    0x7ffeefbff598: 0x000000010001f7a8 0x00007ffeefbff5b0
//    (lldb) po 0x0000000100509d10
//    4300250384
//
//    (lldb) memory read/4xg 0x0000000100509d10
//    0x100509d10: 0x0000000000636261 0xc0000000100509b4
//    0x100509d20: 0x00007fff9b7cd3f8 0x0000000200000001

    //truck.m_name 的abc 存放的地址是0x0000000100509d10
    
}
```






----------
>  行者常至，为者常成！


