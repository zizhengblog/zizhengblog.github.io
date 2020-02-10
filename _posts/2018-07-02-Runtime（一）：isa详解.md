---
layout: post
title: "Runtime（一）：isa详解"
date: 2018-07-02
description: "Runtime（一）：isa详解"
tag: 底层原理
---


<h6>
  版权声明：本文为博主原创文章，未经博主允许不得转载。
  <a target="_blank" href="https://jianghuhike.github.io/1872.html">
  原文地址：https://jianghuhike.github.io/1872.html 
  </a>
</h6>




## 目录

- [runtime介绍](#content1)   
- [isa详解](#content2)   



<!-- ************************************************ -->
## <a id="content1"></a>介绍
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
## <a id="content2"></a>isa详解


在arm64架构之前，isa就是一个普通的指针，指向Class对象或者Meta-Class对象。    
从arm64架构之后，isa进行了优化，变成了一个共用体(union)结构，还使用位域来存储更多的信息。
- [参考文章：位域](https://jianghuhike.github.io/1862.html) 
举个位域的例子：
Person.h
```objc
#import <Foundation/Foundation.h>

NS_ASSUME_NONNULL_BEGIN

struct TallRichHandsom{
    BOOL isTall:1;
    BOOL isRich:1;
    BOOL isHandsom:1;
};

@interface Person : NSObject{
    struct TallRichHandsom _tallRichHandsom;
}

-(BOOL)tall;
-(BOOL)rich;
-(BOOL)handsome;

-(void)setTall:(BOOL)tall;
-(void)setRich:(BOOL)rich;
-(void)setHandsome:(BOOL)handsome;
@end

NS_ASSUME_NONNULL_END
```

- [参考文章：C语言共用体](https://jianghuhike.github.io/1863.html)      

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

#   define ISA_BITFIELD                                                        \
      uintptr_t nonpointer        : 1;                                         \
      uintptr_t has_assoc         : 1;                                         \
      uintptr_t has_cxx_dtor      : 1;                                         \
      uintptr_t shiftcls          : 44; /*MACH_VM_MAX_ADDRESS 0x7fffffe00000*/ \
      uintptr_t magic             : 6;                                         \
      uintptr_t weakly_referenced : 1;                                         \
      uintptr_t deallocating      : 1;                                         \
      uintptr_t has_sidetable_rc  : 1;                                         \
      uintptr_t extra_rc          : 8
#   define RC_ONE   (1ULL<<56)
#   define RC_HALF  (1ULL<<7)

# else
#   error unknown architecture for packed isa
# endif
```



----------
>  行者常至，为者常成！


