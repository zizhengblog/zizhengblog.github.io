---
layout: post
title: "Runtime（一）：isa详解"
date: 2018-07-02
description: "Runtime（一）：isa详解"
tag: Objective-C
---







## 目录

- [runtime介绍](#content1)   
- [isa的数据结构](#content2)   
- [isa的数据查看](#content3)   



<!-- ************************************************ -->
## <a id="content1"></a>runtime介绍
一、Objective-C是一门动态性比较强的编程语言，跟C/C++等语言有着很大的不同。 

编程语言都要经过 编码->预编译->编译->链接->运行，几个阶段，C/C++在运行中的实际结果与编译阶段基本相同，而OC可以在运行过程中做许多事情。         
OC的动态性表现在如下几个方面：     
1、编码阶段调用test方法，实际运行时可能调用的test2方法或者其他类对象的方法。（方法交换）。     
2、运行过程中创建新的类、类对象、类方法。     
3、查看私有变量、私有方法。（比如系统类、SDK内的类）。     

举个例子：
```c
void test(void){
    printf("test");
}

int main(int argc, const char * argv[]) {
    @autoreleasepool {
        //C语言在运行时调用的是test方法。
        test();

        //运行时真正调用的有可能不是person对象的test方法。
        Person * person = [[Person alloc] init];
        [perosn test];
    }
    return 0;
}
```

二、Objective-C的动态性是由Runtime API来支撑的，Runtime API提供的接口基本都是C语言的，源码有C/C++/汇编语言实现。
```oc
//使用runtime提供的接收是需要引入以下头文件
#import <objc/runtime.h>
```

<!-- ************************************************ -->
## <a id="content2"></a>isa的数据结构


在arm64架构之前，isa就是一个普通的指针，指向Class对象或者Meta-Class对象。    
从arm64架构之后，isa进行了优化，变成了一个共用体(union)结构，还使用位域来存储更多的信息。此时要想得到Class或Meta-Class的地址需要&ISA_MASK。     
- [参考文章：位域](https://jianghuhike.github.io/1862.html)     

举个位域使用的例子：     
Person.h
```objc
#import <Foundation/Foundation.h>

NS_ASSUME_NONNULL_BEGIN

-(BOOL)tall;
-(BOOL)rich;
-(BOOL)handsome;

-(void)setTall:(BOOL)tall;
-(void)setRich:(BOOL)rich;
-(void)setHandsome:(BOOL)handsome;
@end

NS_ASSUME_NONNULL_END
```

Person.m
```objc
#import "Person.h"

struct TallRichHandsome{
    BOOL isTall:1;
    BOOL isRich:1;
    BOOL isHandsome:1;
};

@interface Person(){
    struct TallRichHandsome _tallRichHandsome;
}

@implementation Person

-(BOOL)tall{
    //二进制位转为bool值时最好用两次!!转下，一个二进制位转为字节时有可能会出现问题
    //比如0b1,转为一个字节时为 0b11111111,
    return !!_tallRichHandsome.isTall;
}

-(BOOL)rich{
    return !!_tallRichHandsome.isRich;
}

-(BOOL)handsome{
    return !!_tallRichHandsome.isHandsome;
}

-(void)setTall:(BOOL)tall{
    _tallRichHandsome.isTall = tall;
}

-(void)setRich:(BOOL)rich{
    _tallRichHandsome.isRich = rich;
}

-(void)setHandsome:(BOOL)handsome{
    _tallRichHandsome.isHandsome = handsome;
}
@end
```

- [参考文章：C语言共用体](https://jianghuhike.github.io/1863.html) 

举个共用体使用的例子：     
Person.h   
```objc
#import <Foundation/Foundation.h>

NS_ASSUME_NONNULL_BEGIN
@interface Person : NSObject

-(BOOL)tall;
-(BOOL)rich;
-(BOOL)handsome;

-(void)setTall:(BOOL)tall;
-(void)setRich:(BOOL)rich;
-(void)setHandsome:(BOOL)handsome;
@end

NS_ASSUME_NONNULL_END
```

Person.m
```objc
#import "Person.h"

#define PersonTallMask      (1<<0)
#define PersonRichMask      (1<<1)
#define PersonHandsomeMask  (1<<2)

union TallRichHandsome{
    char bits;
    struct{
        BOOL isTall:1;
        BOOL isRich:1;
        BOOL isHandsom:1;
    };
};

@interface Person(){
    union TallRichHandsome _tallRichHandsome;
}

@end

@implementation Person

-(BOOL)tall{
    //二进制位转为bool值时最好用两次!!转下，一个二进制位转为字节时有可能会出现问题
    //比如0b1,转为一个字节时为 0b11111111,
    return !!(_tallRichHandsome.bits & PersonTallMask);
}

-(BOOL)rich{
    return !!(_tallRichHandsome.bits & PersonRichMask);
}

-(BOOL)handsome{
    return !!(_tallRichHandsome.bits & PersonHandsomeMask);
}

-(void)setTall:(BOOL)tall{
    if (tall) {
        _tallRichHandsome.bits |= PersonTallMask;
    }else{
        _tallRichHandsome.bits &= ~PersonTallMask;
    }
}

-(void)setRich:(BOOL)rich{
    if (rich) {
        _tallRichHandsome.bits |= PersonRichMask;
    }else{
        _tallRichHandsome.bits &= ~PersonRichMask;
    }
}

-(void)setHandsome:(BOOL)handsome{
    if (handsome) {
        _tallRichHandsome.bits |= PersonHandsomeMask;
    }else{
        _tallRichHandsome.bits &= ~PersonHandsomeMask;
    }
}
```


通过 iOS_objc4-756.2 源码 我们来看下isa指针的结构。     
- [苹果官方开源代码列表](https://opensource.apple.com/tarballs/)   

我们将部分源码摘抄出来，看下数据结构：
```objc
@interface NSObject <NSObject> {
    Class isa;
}

typedef struct objc_class *Class;

struct objc_class : objc_object {
    // Class ISA;
    Class superclass;
    cache_t cache;             // formerly cache pointer and vtable
    class_data_bits_t bits;    // class_rw_t * plus custom rr/alloc flags
}

struct objc_object {
private:
    isa_t isa;
}

union isa_t {
    isa_t() { }
    isa_t(uintptr_t value) : bits(value) { }

    Class cls;
    uintptr_t bits;
#if defined(ISA_BITFIELD)
    struct {
        ISA_BITFIELD;  // defined in isa.h
    };
#endif
};

# if __arm64__
#   define ISA_MASK        0x0000000ffffffff8ULL
#   define ISA_MAGIC_MASK  0x000003f000000001ULL
#   define ISA_MAGIC_VALUE 0x000001a000000001ULL
#   define ISA_BITFIELD                                                      \
      uintptr_t nonpointer        : 1;                                       \
      uintptr_t has_assoc         : 1;                                       \
      uintptr_t has_cxx_dtor      : 1;                                       \
      uintptr_t shiftcls          : 33; /*MACH_VM_MAX_ADDRESS 0x1000000000*/ \
      uintptr_t magic             : 6;                                       \
      uintptr_t weakly_referenced : 1;                                       \
      uintptr_t deallocating      : 1;                                       \
      uintptr_t has_sidetable_rc  : 1;                                       \
      uintptr_t extra_rc          : 19
#   define RC_ONE   (1ULL<<45)
#   define RC_HALF  (1ULL<<18)
```

我们看到isa指针的数据结构是一个共用体isa_t,我们将源码精简下变为下面这样
```objc
union isa_t {
    Class cls;
    uintptr_t bits;
    struct {
      uintptr_t nonpointer        : 1;                                       
      uintptr_t has_assoc         : 1;                                       
      uintptr_t has_cxx_dtor      : 1;                                       
      uintptr_t shiftcls          : 33; /*MACH_VM_MAX_ADDRESS 0x1000000000*/ 
      uintptr_t magic             : 6;                                       
      uintptr_t weakly_referenced : 1;                                       
      uintptr_t deallocating      : 1;                                       
      uintptr_t has_sidetable_rc  : 1;                                       
      uintptr_t extra_rc          : 19 
    };
};
```
nonpointer     
0，代表普通的指针，存储着Class或者Meta-Class对象的内存地址。     
1，代表优化过，采用位域存储更多的信息。      

has_assoc     
是否有设置过关联对象，如果没有释放时更快。     

has_cxx_dtor     
是否有C++的析构函数（.cxx_destruct）,如果没有释放时更快。

shiftcls      
存储着Class、Meta-Class对象的内存地址信息。

magic    
用于在调试时分辨对象是否未完成初始化

weakly_referenced      
是否有被弱引用指向过，如果没有，释放时更快。

deallocating      
对象是否正在释放 

has_sidetable_rc         
引用计数器是否过大无法存储在isa中，如果为1，那么引用计数会存储在一个叫SideTable的类的属性中。

extra_rc     
里面存储的值是引用计数器减1


<!-- ************************************************ -->
## <a id="content3"></a>isa的数据查看
代码如下
```objc
Person * person = [[Person alloc] init];
NSLog(@"%p",object_getClass(person));
```

日志及lldb调试结果：
```objc
//Person类对象的地址
0x10082d930

(lldb) p/x person->isa
(Class) $1 = 0x000001a10082d935 Person
```

我们先来验证下isa & ISA_MASK 能否得到正确的类对象的地址0x10082d930
```objc
//由上边的源码可知
#define ISA_MASK 0x0000000ffffffff8ULL

(lldb) p/x 0x000001a10082d935 & 0x0000000ffffffff8ULL
(unsigned long long) $7 = 0x000000010082d930

//可以看出我们得到了正确的类对象的地址
```

下面我们把isa的值0x000001a10082d935拆开来看看每一个bit位的情况
```objc
0x35:0011 0101    
0xd9:1101 1001    
0x82:1000 0010    
0x00:0000 0000    
0xa1:1010 0001    
0x01:0000 0001    
0x00:0000 0000    
0x00:0000 0000 
```   

|成员变量|位域|占用位的值|说明|
nonpointer          | 1  | 1 |优化过的isa指针|     
has_assoc           | 1  | 0 |没有设置关联对象|  
has_cxx_dtor        | 1  | 1 |有C++的析构函数|   
shiftcls            | 33 | 0001 0000 0000 1000 0010 1101 1001 0011 0   |类对象地址|    
magic               | 6  | 01 1010|用于在调试时分辨对象是否未完成初始化|                                
weakly_referenced   | 1  | 0 |没有弱引用|                                       
deallocating        | 1  | 0 |没有正在释放|                                      
has_sidetable_rc    | 1  | 0 |引用计数存储在isa中|                                      
uintptr_t extra_rc  | 19 | 0000 0000 0000 0000 000   |该数代表引用计数减1，可知引用计数为1|




来重点看下shiftcls这个成员变量其值为：0001 0000 0000 1000 0010 1101 1001 0011 0             
我们将其最右边的三个bit位补上零后得到：0001 0000 0000 1000 0010 1101 1001 0011 0000           
```objc
(lldb) p/x 0b000100000000100000101101100100110000
(long) $9 = 0x000000010082d930
//我们看到这个值正好是Person类对象的值
//与isa & ISA_MASK的结果是一致的。
```


----------
>  行者常至，为者常成！


