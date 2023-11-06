---
layout: post
title: "OC概要"
date: 2023-11-01
tag: Overview
---





## 目录


- [对象的本质](#content1)  
 - [kvc/kvo](#content2)  
- [Category](#content3)   
- [block](#content4)   
- [Runtime](#content5)  



<!-- ************************************************ -->
## <a id="content1">对象的本质</a>
三种类型的对象：实例对象、类对象、元类对象

两个重要指针：isa指针、super指针

内存分布

<img src="/images/underlying/oc5.png" alt="img">


<!-- ************************************************ -->
## <a id="content2">kvc/kvo</a>

#### kvc

1、找set方法<br>
2、调用是否允许直接访问变量<br>
3、找成员变量<br>
4、调用setValue:forUndefinedKey抛错<br>
<img src="/images/underlying/oc9.png" alt="img">


1、找get方法<br>
2、调用是否允许直接访问变量<br>
3、找成员变量<br>
4、调用ValueForUndefinedKey抛错<br>
<img src="/images/underlying/oc10.png" alt="img">


#### kvo

派生出一个新的类<br>
kvo_person<br>
重写set方法<br>

<img src="/images/underlying/oc8.png" alt="img">

```Objc
NSKeyValueObservingOptions options = NSKeyValueObservingOptionNew | NSKeyValueObservingOptionOld;
[self.person1 addObserver:self forKeyPath:@"age" options:(options) context:@"123"];

-(void)observeValueForKeyPath:(NSString *)keyPath
                     ofObject:(id)object
                       change:(NSDictionary<NSKeyValueChangeKey,id> *)change
                      context:(void *)context{

}

-(void)dealloc{
    //一定要移除监听，否则引起内存泄漏
    [self.person1 removeObserver:self forKeyPath:@"age"];
}
```

```Objc
#import "NSKVONotifying_MJPerson.h"

@implementation NSKVONotifying_Person

- (void)setAge:(int)age{
    _NSSetIntValueAndNotify();
}

// 伪代码
void _NSSetIntValueAndNotify(){
    [self willChangeValueForKey:@"age"];
    [super setAge:age];
    [self didChangeValueForKey:@"age"];
}

- (void)didChangeValueForKey:(NSString *)key{
    // 通知监听器，某某属性值发生了改变
    [oberser observeValueForKeyPath:key ofObject:self change:nil context:nil];
}

// 屏幕内部实现，隐藏了NSKVONotifying_MJPerson类的存在
- (Class)class{
    return [MJPerson class];
}

- (void)dealloc{
    // 收尾工作
}

- (BOOL)_isKVOA{
    return YES;
}

@end
```

<!-- ************************************************ -->
## <a id="content3">Category</a>

#### +Load方法
程序启动加载类的时候，<br>
只调用一次，直接地址调用<br>
先父类再子类，先父类分类再子类分类<br>

#### +initialize方法：
类第一次收到消息的时候调用，<br>
至少调用一次，msg_send方式调用<br>
先父类在子类，分类会覆盖类的initialize方法<br>

#### 关联对象：
全局的hash表



<!-- ************************************************ -->
## <a id="content4">block</a>

### 介绍

#### block：匿名函数，可以作为参数和返回值
```objc
int(^blk)(int a, NSString* str) = ^int(int a, NSString* str){  
    return 10;
};
```
	
#### 变量捕获
被捕获的变量在block内无法修改<br>
要修改：__block int a = 10 修饰


### block的本质

<img src="/images/memory/block1.png" alt="img">

#### block的本质是对象
impl<br>
Desc<br>
捕获的变量<br>

```objc
struct __main_block_impl_0 {
  struct __block_impl impl;
  struct __main_block_desc_0* Desc;
  int age;
  __main_block_impl_0(void *fp, struct __main_block_desc_0 *desc, int _age, int flags=0) : age(_age) {
    impl.isa = &_NSConcreteStackBlock;
    impl.Flags = flags;
    impl.FuncPtr = fp;
    Desc = desc;
  }
};

struct __block_impl {
  void *isa;
  int Flags;
  int Reserved;
  void *FuncPtr;
};

static void __main_block_func_0(struct __main_block_impl_0 *__cself) {
    int age = __cself->age; // bound by copy
    
    printf("Block\n age=%d\n",age);
}
```

#### block的调用本质是方法
impl下有一个FuncPtr指针<br>
block初始化时传入方法地址<br>
要清楚方法的参数是如何传递的<br>
```objc
int main(int argc, char * argv[]) {

        int age = 10;

        void(*blk)(void)=&__main_block_impl_0(
                                              __main_block_func_0,
                                              &__main_block_desc_0_DATA,
                                              age));

        age = 20;

       ((__block_impl *)blk)->FuncPtr)(blk);
    }
    return 0;
}
```


#### 捕获的变量

自动变量：值捕获 int / Person

静态局部变量：地址捕获
 
不会捕获全局变量


### block的类型

#### 一、block的类型（block是对象）

没有捕获auto变量：__NSGlobalBlock__:数据段<br>
copy操作无反应

捕获auto变量：__NSStackBlock__:栈区<br>
copy操作复制到堆空间

__NSStackBlock__执行copy后： __NSMallocBlock__:堆区<br>
copy操作引用计数加1


#### 二、block的copy

作为返回值会执行copy操作

强指针指向会执行copy操作

usingBlock中做参数会执行copy操作

GCD中做参数会执行copy操作




#### 三、捕获对象类型自动变量
1、如果Block在栈上，不会对对象类型auto变量产生强引用。<br>
2、如果Block在堆上，<span style="color:red;font-weight:bold">会根据对象类型auto变量的修饰符</span> __strong、 __weak、 __unsafe_unretained
做出相应的操作，形成强引用或若引用。



#### 四、desc下的copy方法和dispose方法
Block被拷贝到堆上会调用Block内部的copy函数，对捕获的变量强弱引用<br>
Block从堆上移除会调用Block内部的dispose函数，释放捕获的变量


### __block原理

#### 一、_ _block 修饰为什么就能在block内部修改值了

```objc
__block int age = 10;
blk b=^(){
    age = 20;
};
```

——block修饰的age会变成一个对象block_age<br>
对象内部有一个属性age<br>
对象block_age会被block捕获<br>
block内修改的是block_age下的age属性而不是<br>
block_age对象<br>

```objc
typedef void(*blk)(void);
struct __Block_byref_age_0 {
    void *__isa;
    __Block_byref_age_0 *__forwarding;
    int __flags;
    int __size;
    int age;
};

struct __main_block_impl_0 {
    struct __block_impl impl;
    struct __main_block_desc_0* Desc;
    __Block_byref_age_0 *age; // by ref
    __main_block_impl_0(void *fp, struct __main_block_desc_0 *desc, __Block_byref_age_0 *_age, int flags=0) : age(_age->__forwarding) {
        impl.isa = &_NSConcreteStackBlock;
        impl.Flags = flags;
        impl.FuncPtr = fp;
        Desc = desc;
    }
};


static void __main_block_func_0(struct __main_block_impl_0 *__cself) {
    __Block_byref_age_0 *age = __cself->age; // bound by ref
    
    (age->__forwarding->age) = 20;
}
```
#### 二、forwarding指针

<img src="/images/memory/block7.png" alt="img">

这就是为什么是 (age->__forwarding->age) = 20;而不是(age->age) = 20;的原因


### block的循环引用

```objc
Person * person = [[Person alloc] init];
person.blk = {
    person.name
}
```
<img src="/images/memory/block8.png" alt="img">

```objc
Person * person = [[Person alloc] init];
__weak typeof(person) weakPerson = person;
person.blk = {
    person.name
}
```
<img src="/images/memory/block9.png" alt="img">



<!-- ************************************************ -->
## <a id="content5">Runtime</a>

### class的结构
<img src="/images/underlying/oc16.png" alt="img">


#### 一、isa的结构
```objc
union isa_t {
    Class cls;
    uintptr_t bits;
    struct {
      uintptr_t nonpointer        : 1;  //是否是优化过的指针                                     
      uintptr_t has_assoc         : 1;  //是否有关联对象                                     
      uintptr_t has_cxx_dtor      : 1;                                       
      uintptr_t shiftcls          : 33; /*MACH_VM_MAX_ADDRESS 0x1000000000*/ 
      uintptr_t magic             : 6;                                       
      uintptr_t weakly_referenced : 1;  //是否有弱引用                                     
      uintptr_t deallocating      : 1;                                       
      uintptr_t has_sidetable_rc  : 1;  //是否有引用计数表                                     
      uintptr_t extra_rc          : 19  //引用计数
    };
};
```

#### 二、cache的结构
<img src="/images/underlying/oc22.png" alt="img">


#### 三、bits的结构
二维数组
<img src="/images/underlying/oc17.png" alt="img">

<img src="/images/underlying/oc19.png" alt="img">


### 消息机制

1、消息发送是在查找方法实现<br>
2、动态方法解析是给类一个机会自己动态生成对应的方法<br>
3、消息转发是当前类没有处理看看其它的类能不能处理<br>


#### 一、消息发送
```objc
objc_msgSend(person, sel_registerName("test"));   
消息接收者（receiver）：person    
消息名称：test 
```

<img src="/images/underlying/msgsend1.png" alt="img">


#### 二、动态方法解析
<img src="/images/underlying/msgsend2.png" alt="img">
```objc
+(BOOL)resolveInstanceMethod:(SEL)sel{
    if (sel == @selector(test)) {
        
        NSLog(@"resolveInstance");
        
        //获取method
        Method method = class_getInstanceMethod(self, @selector(other));
        
        //动态添加方法
        class_addMethod(self,
                        sel,
                        method_getImplementation(method),
                        method_getTypeEncoding(method));
        
        //
        return YES;
    }
    
    return [super resolveInstanceMethod:sel];
}
```

#### 三、消息转发
<img src="/images/underlying/msgsend3.png" alt="img">
```objc
-(id)forwardingTargetForSelector:(SEL)aSelector{
    NSLog(@"%s",__func__);
    if (aSelector == @selector(test)) {
        Class Cat = NSClassFromString(@"Cat");
        
        //返回一个实例对象 就会调用实例对象的test方法
        //return [[Cat alloc] init];
        
        //返回nil 就会调用methodSignatureForSelector
        return nil;
    }
    return [super forwardingTargetForSelector:aSelector];
}


- (NSMethodSignature *)methodSignatureForSelector:(SEL)aSelector{
    NSLog(@"%s",__func__);
    
    if (aSelector == @selector(test)) {
        //注册消息转发到的方法的类型
        return [NSMethodSignature signatureWithObjCTypes:"i20@0:8i:4"];
    }
    
    return [super methodSignatureForSelector:aSelector];
}


- (void)forwardInvocation:(NSInvocation *)anInvocation{
    NSLog(@"%s",__func__);
    
    //anInvocation 包含selector targe argument
    if (anInvocation.selector == @selector(test)) {
        
        Class Cat = NSClassFromString(@"Cat");
        id cat = [[Cat alloc] init];
        
        //更改消息接收者
        anInvocation.target =cat;
        //更改selector
        anInvocation.selector = @selector(catTest:);
        //设置新方法参数
        int age = 5;
        [anInvocation setArgument:&age atIndex:2];
        
        //新方法调用
        [anInvocation invoke];
        
        //获取新方法的返回值
        int reValue;
        [anInvocation getReturnValue:&reValue];
        NSLog(@"reValue=%d",reValue);
    }
}
@end
```

### 几个知识点

#### 一、@synthesize 和 @dynamic
```objc
@synthesize的作用就是通知编译器为相关属性自动生成，成员变量、set、get方法。
@synthesize age=_age,height=_height;

@dynamic是告诉编译器不用自动生成成员变量、set、get方法，等到运行时再添加方法实现。
@dynamic age;
```

#### 二、Super的原理
```objc
[super test];
static void _I_Student_test(Student * self, SEL _cmd) {
    struct __rw_objc_super arg = {
        self,//消息的接收者还是self
        self.superClass//查找从父类开始
    }
    objc_msgSendSuper(arg,sel_registerName("test"));
}

[self class] //Student
[self superclass]//Person
[super  class]);//Student
[super superclass]//Person
```

#### 三、isMemberOfClass 和 isKindOfClass
```objc
NSLog(@"%d",[person isMemberOfClass:[Person class]]);//1
NSLog(@"%d",[person isMemberOfClass:[NSObject class]]);//0
NSLog(@"%d",[person isKindOfClass:[Person class]]);//1
NSLog(@"%d",[person isKindOfClass:[NSObject class]]);//1

NSLog(@"%d",[NSObject isMemberOfClass:[NSObject class]]);//0
NSLog(@"%d",[NSObject isKindOfClass:[NSObject class]]);//1
NSLog(@"%d",[Person isMemberOfClass:[Person class]]);//0
NSLog(@"%d",[Person isKindOfClass:[Person class]]);//0

isMemberOf不会顺着super指针向上查找
isKindOf会顺着super向上查找
```

#### 四、cls指针
```objc
-(void)print{
    NSLog(@"my name is %@",self.name);
}

- (void)viewDidLoad {
    [super viewDidLoad];
    id cls = [Person class];//cls就是isa指针
    void * obj = &cls;//obj就是实例对象
    // 是否调用成功，打印什么？
    [(__bridge id)obj print];//栈空间的内存分布
}
调用成功打印
my name is <ViewController: 0x100704080>
```


### runtime的应用

#### 一、遍历查看所有的成员变量和属性
```
1.textfiled修改占位文字的颜色
2.字典转模型：
    for循环拿到属性转为字符串作为key
     通过key,dic[key]取出value
     通过kvc,setValueForKey(key,value)对属性赋值
3.自动归档解档
     自定义类型存数据库需要归解档
     for循环拿到属性转为字符串作为key
     归档:通过kvc拿到value，调用encode(key,value)归档
     解档:value = decode(key),然后kvc,setValueForKey(key,value)
```

#### 二、交换方法实现
```
1.交换数组的insertObject方法，解决索引越界崩溃问题
```

#### 三、消息转发解决方法找不到的崩溃问题
```
实现methodSignature方法
实现forwardInvocation方法,方法内可以做一些提示或上送操作
```

#### 四、利用关联对象给分类添加属性

----------
>  行者常至，为者常成！


