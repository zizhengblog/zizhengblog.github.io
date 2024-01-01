---
layout: post
title: "1、Singleton"
date: 2017-06-01
description: ""
tag: 设计模式
---






## 目录

* [什么是单例模式](#content1)
* [单例模式实现](#content2)
* [单例模式应用](#content3)
* [补充](#content4)







<!-- ************************************************ -->
## <a id="content1"></a>什么是单例模式

最简单，最常用的设计模式之一，主要解决了一个全局使用的类频繁地创建与销毁。   
当您想控制实例数目，节省系统资源的时候，就可以使用单例模式   

1、全局唯一，只初始化一次，只有一个实例      
2、只提供一个接口，私有化初始化接口      
3、线程安全      
4、最好能懒加载（不强求） 


<!-- ************************************************ -->
## <a id="content2"></a>单例模式实现

**一、实现**

```swift
class Singleton: NSObject {

    //存储类型属性在初始化时必须赋值，因为没有初始化器来初始化它，
    //存储类型属性的赋值是线程安全的，由系统 保证
    //let保证只赋值一次
    //存储类型属性默认lazy只有再使用时才会初始化
    //以上保证了可以实现单例模式
    static let shared = Singleton()
    
    //私有化初始化器
    private override init() {
        super.init()
    }
}
```

**二、唯一性测试**

```swift
func test1() {
    for _ in 0..<10 {
        let sle = Singleton.shared
        print(sle)
    }
    
    
    //调用初始化方法报错：Singleton' initializer is inaccessible due to 'private' protection level
    //let sle = Singleton()

    /**
    <LCClientDemo.Singleton: 0x281690090>
    <LCClientDemo.Singleton: 0x281690090>
    <LCClientDemo.Singleton: 0x281690090>
    <LCClientDemo.Singleton: 0x281690090>
    <LCClientDemo.Singleton: 0x281690090>
    <LCClientDemo.Singleton: 0x281690090>
    <LCClientDemo.Singleton: 0x281690090>
    <LCClientDemo.Singleton: 0x281690090>
    <LCClientDemo.Singleton: 0x281690090>
    <LCClientDemo.Singleton: 0x281690090>
    */
}
```

**三、线程安全测试**

```swift
func test2() {
    for _ in 0...10 {
        let thread = Thread.init {
            let sle = Singleton.shared
            print(Thread.current, sle)
        }
        thread.start()
    }
}

/**
<NSThread: 0x2801c0340>{number =  8, name = (null)} <LCClientDemo.Singleton: 0x281690090>
<NSThread: 0x2801c15c0>{number =  9, name = (null)} <LCClientDemo.Singleton: 0x281690090>
<NSThread: 0x2801c1c80>{number = 10, name = (null)} <LCClientDemo.Singleton: 0x281690090>
<NSThread: 0x2801c09c0>{number = 11, name = (null)} <LCClientDemo.Singleton: 0x281690090>
<NSThread: 0x2801c0000>{number = 12, name = (null)} <LCClientDemo.Singleton: 0x281690090>
<NSThread: 0x2801c2080>{number = 14, name = (null)} <LCClientDemo.Singleton: 0x281690090>
<NSThread: 0x2801c5880>{number = 17, name = (null)} <LCClientDemo.Singleton: 0x281690090>
<NSThread: 0x2801c7f40>{number = 18, name = (null)} <LCClientDemo.Singleton: 0x281690090>
<NSThread: 0x2801c0c80>{number = 13, name = (null)} <LCClientDemo.Singleton: 0x281690090>
<NSThread: 0x2801c0940>{number = 15, name = (null)} <LCClientDemo.Singleton: 0x281690090>
<NSThread: 0x2801c1680>{number = 16, name = (null)} <LCClientDemo.Singleton: 0x281690090>
*/
```





<!-- ************************************************ -->
## <a id="content3"></a>单例应用

1、程序整个生命周期内有且只有一份   
    UIApplication.shared

2、管理工具类    
    UserDefaults.standard   
    FileManager.default    
    NotificationCenter.default    


<!-- ************************************************ -->
## <a id="content4"></a>补充

**一、OC的实现方式**

```
static LCLoginMananger * manager = nil;
+(instancetype)shareInstance{
    static dispatch_once_t onceToken;

    //保证线程安全和一次执行
    dispatch_once(&onceToken, ^{
        manager = [[LCLoginMananger alloc] init];
        manager.biometry = [[LCBiometryTool alloc] init];
        [manager copyBundleToUserDefault];
    });
    return manager;
}


//保证alloc和new出的也是同一对象
//+(instancetype)alloc{
//    if (!manager) {
//        manager = [super alloc];
//    }
//    return manager;
//}


//保证allocWithZone出的也是同一个对象
//alloc会调用allocWithZone，所以重写了这个方法，上边的方法可以不重写
+(instancetype)allocWithZone:(struct _NSZone *)zone{
    if (!manager){
        manager = [super allocWithZone:zone];
    }
    return manager;
}
```

**二、调用**

```
-(void)sharedTest{

    for (int i = 0; i<10; i++) {
        
        NSThread * thread = [[NSThread alloc] initWithBlock:^{
            LCLoginMananger * manger = [LCLoginMananger shareInstance];
            NSLog(@"manager = %@,thread = %@",manger,[NSThread currentThread]);
        }];
        
        [thread start];
    }
    
    
    LCLoginMananger * manager0 = [[LCLoginMananger alloc] init];
    NSLog(@"manager0 = %@",manager0);
    
    LCLoginMananger * manager1 = [LCLoginMananger new];
    NSLog(@"manager1 = %@",manager1);
    
    
    LCLoginMananger * manager2 = [[LCLoginMananger allocWithZone:nil] init];
    NSLog(@"manager2 = %@",manager2);
    
    /**
     manager0 = <LCLoginMananger: 0x2813f9a60>
     manager1 = <LCLoginMananger: 0x2813f9a60>
     manager2 = <LCLoginMananger: 0x2813f9a60>
     manager  = <LCLoginMananger: 0x2813f9a60>,thread = <NSThread: 0x28048d080>{number = 9, name = (null)}
     manager  = <LCLoginMananger: 0x2813f9a60>,thread = <NSThread: 0x28048d0c0>{number = 10, name = (null)}
     manager  = <LCLoginMananger: 0x2813f9a60>,thread = <NSThread: 0x28048cf40>{number = 11, name = (null)}
     manager  = <LCLoginMananger: 0x2813f9a60>,thread = <NSThread: 0x28048d040>{number = 12, name = (null)}
     manager  = <LCLoginMananger: 0x2813f9a60>,thread = <NSThread: 0x28048d000>{number = 13, name = (null)}
     manager  = <LCLoginMananger: 0x2813f9a60>,thread = <NSThread: 0x28048cf80>{number = 15, name = (null)}
     manager  = <LCLoginMananger: 0x2813f9a60>,thread = <NSThread: 0x28048cfc0>{number = 14, name = (null)}
     manager  = <LCLoginMananger: 0x2813f9a60>,thread = <NSThread: 0x28048ce40>{number = 16, name = (null)}
     manager  = <LCLoginMananger: 0x2813f9a60>,thread = <NSThread: 0x28048ce80>{number = 17, name = (null)}
     manager  = <LCLoginMananger: 0x2813f9a60>,thread = <NSThread: 0x28048d100>{number = 18, name = (null)}
     */
}
```


----------
>  行者常至，为者常成！


