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

- [Runtime](#content)  

- [](#content)   
<!-- ************************************************ -->
## <a id="content"></a>



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




----------
>  行者常至，为者常成！


