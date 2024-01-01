---
layout: post
title: "copy与mutableCopy"
date: 2018-04-04
description: "copy与mutableCopy"
tag: Objective-C
---






## 目录
- [copy与mutableCopy](#content1)   
- [copy修饰符](#content2)   
- [自定义对象实现copy方法](#content3)   




<!-- ************************************************ -->
## <a id="content1"></a>copy与mutableCopy

浅拷贝：简单的指针拷贝，不会产生新的对象。      
深拷贝：内容拷贝，会产生的新对象。      

代码段一：

```objc
NSString * str1 = [NSString stringWithFormat:@"test"];
NSString * str2 = [str1 copy];//浅拷贝
NSMutableString * str3 = [str1 mutableCopy];//深拷贝
[str3 appendString:@"123"];

NSLog(@"%@,%@,%@",str1,str2,str3);
NSLog(@"%p,%p,%p",str1,str2,str3);
```
控制台
```objc
test,test,test123
0xbf95750821a0a1f4,0xbf95750821a0a1f4,0x28374f870
```

指针指向如下图所示

<img src="/images/underlying/other4.png" alt="img">


代码段二：

```objc
NSMutableString * str1 = [NSMutableString stringWithFormat:@"test"];
NSString * str2 = [str1 copy];//深拷贝
NSMutableString * str3 = [str1 mutableCopy];//深拷贝
[str3 appendString:@"123"];

NSLog(@"%@,%@,%@",str1,str2,str3);
NSLog(@"%p,%p,%p",str1,str2,str3);
```

控制台
```objc
test,test,test123
0x281074840,0xb5303a62f7c8f1df,0x281074a20
```

指针指向如下图所示

<img src="/images/underlying/other5.png" alt="img">

总结

<img src="/images/underlying/other6.png" alt="img">


<!-- ************************************************ -->
## <a id="content2"></a>copy修饰符

**一、看下面代码**

```objc
@interface ViewController ()
@property (nonatomic, copy) NSMutableArray * array;
@end

@implementation ViewController

- (void)viewDidLoad{
    [super viewDidLoad];
    
    NSMutableArray * array = [NSMutableArray array];
    NSLog(@"array = %p",array);
    self.array = array;
    NSLog(@"self.array = %p",self.array);
    [self.array addObject:@"123"];
}

@end
```

控制台日志
```objc
array = 0x2835ee2e0
self.array = 0x1c6573400
-[__NSArray0 addObject:]: unrecognized selector sent to instance 0x1c6573400
```
程序崩溃了。

我们注意到array 跟self.array 指向的不是同一个对象，因为属性是copy，所以在赋值给self.array时进行了copy。其set方法的实现如下：
```objc
-(void)setArray:(NSMutableArray *)array{
    if (_array!=array) {
        [_array release];
        _array = [array copy];
    }
}
```
虽然array是可变数组，但执行完copy后变为了不可变数组，所以调用addObject:方法程序会报unrecognized selector to instance xxxx的错误。

<span style="color:red">所以copy不要修饰可变对象，比如NSMutableString、 NSMutableArray、 NSMutableDictionary</span>


**二、copy与strong**

使用strong时，地址相同

```objc
@interface ViewController ()
@property (nonatomic, strong) NSArray * array;
@end

@implementation ViewController
- (void)viewDidLoad{
    [super viewDidLoad];
    
    NSMutableArray * array = [NSMutableArray array];
    NSLog(@"array = %p",array);
    self.array = array;
    NSLog(@"self.array = %p",self.array);
}
```
控制台日志
```objc
array = 0x283efa820
self.array = 0x283efa820
```

使用copy时 地址不同
```objc
@interface ViewController ()
@property (nonatomic,   copy) NSArray * array;
@end

@implementation ViewController

- (void)viewDidLoad{
    [super viewDidLoad];
    
    NSMutableArray * array = [NSMutableArray array];
    NSLog(@"array = %p",array);
    self.array = array;
    NSLog(@"self.array = %p",self.array);
}

@end
```

控制台日志 地址不相同
```objc
array = 0x2822f70f0
self.array = 0x1c6573400
```

将上面的两段代码中的         
`NSMutableArray * array = [NSMutableArray array];`      
换成     
`NSArray * array = [NSArray array];`     
结果又会怎样呢？可以编写代码试试


**三、copy的应用**

```objc
UITextField * textField = [[UITextField alloc] init];
textField.text = @"test";
```

如上面的代码我们看下text属性,使用了copy
```objc
@property(nullable, nonatomic,copy)   NSString * text;                 // default is nil
```

为什么使用copy?     
1、如果外边传进来的是NSMutableString的实例，那么无论外边怎么修改，都不会影响text的显示。
```objc
NSMutableString * mString = [NSMutableString stringWithString:@"123"];
UITextField * textField = [[UITextField alloc] init];

//text使用copy
//textField.text = [mString copy];产生了一个新对象并且不可变，mString的修改不影响text的显示
textField.text = mString;
```
2、不允许text拼接修改
```objc
//无法这样使用
[textField.text appendString:@""];
```


<!-- ************************************************ -->
## <a id="content3"></a>自定义对象实现copy方法

```objc
@interface Person : NSObject<NSCopying>
@property (nonatomic, copy) NSString * name;
@property (nonatomic, assign) int age;
@end


-(id)copyWithZone:(NSZone *)zone{
    Person * person = [[Person allocWithZone:zone] init];
    person.age = self.age;
    person.name = self.name;
    return person;
}
```

调用
```objc
- (void)viewDidLoad{
    [super viewDidLoad];
    
    Person * person1 = [[Person alloc] init];
    person1.age = 18;
    person1.name = @"小明";
    
    Person*person2 = [person1 copy];
    
    person2.age = 20;
    person2.name = @"tom";
    
    NSLog(@"perosn1=%@,person1.name=%@,person1.age=%d",person1,person1.name,person1.age);
    
    NSLog(@"perosn2=%@,person2.name=%@,person2.age=%d",person2,person2.name,person2.age);

}
```

控制台日志
```objc
perosn1=<Person: 0x2819d3b20>,person1.name=小明,person1.age=18
perosn2=<Person: 0x2819d0f60>,person2.name=tom,person2.age=20
```


----------
>  行者常至，为者常成！


