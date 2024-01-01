---
layout: post
title: "Runtime（八）：API"
date: 2018-03-04
description: "Runtime（八）：API"
tag: Objective-C
---






## 目录
- [类](#content1)   
- [成员变量](#content2)   
- [属性](#content3)  
- [方法](#content4)   
- [具体应用](#content5)   











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
    
    //添加成员变量 (添加成员变量应该在注册类之前)
    class_addIvar(Dog, "_number", 4, 1, @encode(int));
    class_addIvar(Dog, "_age", 4, 1, @encode(int));


    //添加方法(添加方法可在注册类之前也可在之后)
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

<!-- ************************************************ -->
## <a id="content2"></a>成员变量


```objc
//获取成员变量的信息 注意name是属性名 _name 才是成员变量
Ivar nameIvar = class_getInstanceVariable([Person class], "_name");
const char * name   = ivar_getName(nameIvar);
const char * encode = ivar_getTypeEncoding(nameIvar);
NSLog(@"%s,%s",name,encode);
```

```objc
//获取一个类的成员变量列表
unsigned int outCount = 0;
Ivar * iVarList = class_copyIvarList([Person class], &outCount);
NSLog(@"outCount = %d",outCount);//outCount = 2

for (int i=0; i<outCount; i++) {
    Ivar instanceVar = iVarList[i];
    const char * name   = ivar_getName(instanceVar);
    const char * encode = ivar_getTypeEncoding(instanceVar);
    NSLog(@"%s,%s",name,encode);
}
//copy 最后需要释放
free(iVarList);
```

```objc
Person * person = [[Person alloc] init];
person.name = @"小明";

//获取成员变量的值
id personName = object_getIvar(person, nameIvar);
NSLog(@"personName = %@",personName);//personName = 小明

//设置成员变量的值
object_setIvar(person, nameIvar, @"Tom");
personName = object_getIvar(person, nameIvar);
NSLog(@"personName = %@",personName);//personName = Tom
```


<!-- ************************************************ -->
## <a id="content3"></a>属性

```objc
//获取属性信息
objc_property_t nameProperty = class_getProperty([Person class], "name");
const char * name = property_getName(nameProperty);
const char * attribute = property_getAttributes(nameProperty);
NSLog(@"%s,%s",name,attribute);// name,T@"NSString",&,N,V_name
```

```objc
//获取一个类的属性列表
unsigned int outCount;
objc_property_t * propertyList = class_copyPropertyList([Person class], &outCount);
for (int i=0; i<outCount; i++) {
    objc_property_t property = propertyList[i];
    const char * name = property_getName(property);
    const char * attribute = property_getAttributes(property);
    NSLog(@"%s,%s",name,attribute);
}

//最后需要释放
free(propertyList);
```


<!-- ************************************************ -->
## <a id="content4"></a>方法

```objc
//获取方法信息
//class_getClassMethod(Class  _Nullable __unsafe_unretained class, SEL  _Nonnull name);
Method runMethod = class_getInstanceMethod([Person class], @selector(run));
SEL run = method_getName(runMethod);
const char * type = method_getTypeEncoding(runMethod);
IMP imp = method_getImplementation(runMethod);
```

```objc
//获取方法列表
unsigned int  outCount;
//获取对象方法类表
Method * methodList = class_copyMethodList([Person class], &outCount);
//获取类方法类表
//Method * methodList = class_copyMethodList(object_getClass([Person class]), &outCount);
for (int i=0; i<outCount; i++) {
    Method method = methodList[i];
    SEL run = method_getName(method);
    const char * type = method_getTypeEncoding(method);
    IMP imp = method_getImplementation(method);
    NSLog(@"%s,%s",run,type);
}
```

```objc
//动态替换方法
void test(void){
    printf("test\n");
}

- (void)viewDidLoad {
    [super viewDidLoad];
    class_replaceMethod([Person class], @selector(run), (IMP)test, "v");
    Person * person = [[Person alloc] init];
    [person run];
}

class_replaceMethod([Person class], @selector(run), imp_implementationWithBlock(^{
    NSLog(@"123");
}), "v");

[[[Person alloc] init] run];//123
```

```objc
//交换方法
- (void)viewDidLoad {
    [super viewDidLoad];
    
    //交换方法：不仅可以交换类内还可以与其他类交换
    //Method runMethod = class_getInstanceMethod([Person class], @selector(run));
    Method runMethod = class_getInstanceMethod([Student class], @selector(run));
    Method printMethod = class_getInstanceMethod([Person class], @selector(print));

    method_exchangeImplementations(printMethod, runMethod);
    
    Person * person = [[Person alloc] init];
    [person print];//-[Student run]
}
```


<!-- ************************************************ -->
## <a id="content5"></a>具体应用
1、利用关联对象给分类添加属性。

2、遍历类的所有成员变量
- 修改textField的占位文字     
- 字典转模型     
- 自动归档解档    

3、交换方法实现（交换系统方法）

4、利用消息转发机制，解决方法找不到的异常问题。










----------
>  行者常至，为者常成！


