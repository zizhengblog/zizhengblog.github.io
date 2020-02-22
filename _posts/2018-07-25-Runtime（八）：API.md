---
layout: post
title: "Runtime（八）：API"
date: 2018-07-22
description: "Runtime（八）：API"
tag: 底层原理
---


<h6>
  版权声明：本文为博主原创文章，未经博主允许不得转载。
  <a target="_blank" href="https://jianghuhike.github.io/18725.html">
  原文地址：https://jianghuhike.github.io/18725.html 
  </a>
</h6>




## 目录
- [类](#content1)   
- [成员变量](#content2)   






<!-- ************************************************ -->
## <a id="content1"></a>类
```objc
@implementation ViewController

void run(id self,SEL _cmd){
    printf("run\n");
}


-(void)run{
    NSLog(@"run");
}

- (void)viewDidLoad {
    [super viewDidLoad];
    
    Person * person = [[Person alloc] init];
    
    //获取isa指向的类
    Class cls = object_getClass(person);//类对象
    Class cls1 = object_getClass(cls);//元类对象
    Class cls2 = object_getClass([Person class]);//元类对象
    
    //判断一个对象是否是Class
    BOOL isCls  = object_isClass(person);
    BOOL isCls1 = object_isClass([Person class]);
    BOOL isCls2 = object_isClass(object_getClass([person class]));
    
    //判断一个对象是否是meta-class
    BOOL isMetaCls  =  class_isMetaClass(person);
    BOOL isMetaCls1 =  class_isMetaClass([Person class]);
    BOOL isMetaCls2 =  class_isMetaClass(object_getClass([Person class]));
    
    
    //获取父类
    Class cls3 = class_getSuperclass([Person class]);
    
    
    //修改实例对象的isa指向
    object_setClass(person, [Student class]);
    [person run];
    
    
    //动态创建一个类
    Class Dog = objc_allocateClassPair([NSObject class], "Dog", 0);
    
    //添加成员变量
    class_addIvar(Dog, "_number", 4, 1, @encode(int));
    class_addIvar(Dog, "_age", 4, 1, @encode(int));
    //添加方法
    
    //class_addMethod(Dog, @selector(run), (IMP)run, "v@:");//添加c语言方法
    
    IMP methodImp = class_getMethodImplementation([self class], @selector(run));
    class_addMethod(Dog, @selector(run), methodImp, "v@:");
    



    //注册类
    objc_registerClassPair(Dog);
    
    //查看该类的实例对象需要的空间大小
    int size = class_getInstanceSize(Dog);

    //创建一个实例对象
    id dog = [[Dog alloc] init];
    
    [dog setValue:@10 forKey:@"number"];
    [dog setValue:@3 forKey:@"age"];
    
    NSLog(@"number=%@,name=%@",[dog valueForKey:@"number"],[dog valueForKey:@"age"]);
        
    [dog run];

//    objc_disposeClassPair(Dog);//销毁一个类
}
```


----------
>  行者常至，为者常成！


