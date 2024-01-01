---
layout: post
title: "Runtime（十）：应用举例"
date: 2018-03-06
description: "Runtime（十）：应用举例"
tag: Objective-C
---






## 目录
- [修改textField的占位文字](#content1)   
- [字典转模型](#content2)   
- [自动归档解档](#content3)   
- [交换方法实现](#content4)   
- [利用消息转发机制，解决方法找不到的异常问题](#content5)   






<!-- ************************************************ -->
## <a id="content1"></a>修改textField的占位文字

```objc
- (void)viewDidLoad {
    [super viewDidLoad];
    
    UITextField * textField = [[UITextField alloc] initWithFrame:CGRectMake(20, 100, 300, 40)];
    textField.placeholder = @"请输入文字";
    [self.view addSubview:textField];
    
    
    unsigned int outCount;
    Ivar * varList = class_copyIvarList([UITextField class], &outCount);
    for (int i=0; i<outCount; i++) {
        Ivar var = varList[i];
        const char * varName = ivar_getName(var);
        NSLog(@"varName=%s",varName);
    }
    
    //通过遍历成员变量，可以看到有一个_placeholderLabel成员变量
    Ivar ivar =  class_getInstanceVariable([UITextField class], "_placeholderLabel");
    
    //拿到成员变量的值
    UILabel * placeholderLabel = ((UILabel*)object_getIvar(textField, ivar));
    
    placeholderLabel.textColor = [UIColor redColor];
}
```


<!-- ************************************************ -->
## <a id="content2"></a>字典转模型

给NSObject添加分类，在分类里添加方法
```objc

+(instancetype)lc_personWithJson:(NSDictionary*)jsonDic{
    
    id obj = [[self alloc] init];
    
    unsigned int outCount;
    
    objc_property_t * propertyList = class_copyPropertyList([self class], &outCount);
    
    for (int i = 0; i<outCount; i++) {
        objc_property_t property = propertyList[i];
        
        const char * propertyName = property_getName(property);
        
        NSLog(@"propertyName = %s",propertyName);
        
        NSString * key = [NSString stringWithUTF8String:propertyName];
        id value = nil;
        if ([key isEqualToString:@"ID"]) {
            value = jsonDic[@"id"];
        }else{
            value = jsonDic[key];
        }
        
        [obj setValue:value forKey:key];
    }

    free(propertyList);
    
    return obj;
}
```

使用
```objc
- (void)viewDidLoad {
    [super viewDidLoad];
    
    NSDictionary * dic = @{
        @"name":@"xiaoming",
        @"age":@(10),
        @"id":@(123)
    };
    
    Person * person = [Person lc_personWithJson:dic];
}
```


<!-- ************************************************ -->
## <a id="content3"></a>自动归档解档

Person.h文件
```objc
@interface Person : NSObject
@property (nonatomic, strong) NSString * name;
@property (nonatomic, assign) int age;
@property (nonatomic, assign) int ID;
@end
```

Person.m文件
```objc
#import "Person.h"
#import <objc/runtime.h>

@interface Person()<NSCoding>

@end

@implementation Person


- (void)encodeWithCoder:(NSCoder *)coder{
    
    unsigned int outCount;
    objc_property_t * propertyList = class_copyPropertyList([Person class], &outCount);

    for (int i = 0; i<outCount; i++) {
        objc_property_t property = propertyList[i];
        const char * propertyName = property_getName(property);

        NSString * key = [NSString stringWithUTF8String:propertyName];
        id value = [self valueForKey:key];

        [coder encodeObject:value forKey:key];
    }

    free(propertyList);
}


- (instancetype)initWithCoder:(NSCoder *)coder{
    
    if (self = [super init]) {
        unsigned int outCount;
        objc_property_t * propertyList = class_copyPropertyList([Person class], &outCount);

        for (int i = 0; i<outCount; i++) {
            objc_property_t property = propertyList[i];
            const char * propertyName = property_getName(property);

            NSString * key = [NSString stringWithUTF8String:propertyName];
            id value = [coder decodeObjectForKey:key];

            [self setValue:value forKey:key];
        }

        free(propertyList);
    }

    return self;
}
@end
```

使用
```objc
- (void)viewDidLoad {
    [super viewDidLoad];
    
    NSDictionary * dic = @{
        @"name":@"xiaoming",
        @"age":@(10),
        @"id":@(123)
    };
    
    Person * person = [Person lc_personWithJson:dic];
    //序列化
    NSData *archiveCarPriceData = [NSKeyedArchiver archivedDataWithRootObject:person];

    //存数据
    NSUserDefaults * userDefault = [NSUserDefaults standardUserDefaults];
    [userDefault setObject:archiveCarPriceData forKey:@"personkey"];
    [userDefault synchronize];

    //取数据
    NSData * data = [userDefault objectForKey:@"personkey"];

    //反序列化
    Person * person1 = [NSKeyedUnarchiver unarchiveObjectWithData:data];

}
```


<!-- ************************************************ -->
## <a id="content4"></a>交换方法实现

如下代码当我们向数组添加个空元素时程序会报错     

```objc
NSMutableArray * mArray = [NSMutableArray new];
[mArray addObject:nil];
```

reason: '*** -[__NSArrayM insertObject:atIndex:]: object cannot be nil'       

可以看出调用的是`__NSArrayM`类的系统方法`insertObject:atIndex:`,那么我们就可以交换这个方法进行一些处理。    

创建分类：NSMutableArray+category 
```objc
#import "NSMutableArray+category.h"
#import <objc/runtime.h>

@implementation NSMutableArray (category)

+ (void)load{
    
    Class cls = NSClassFromString(@"__NSArrayM");
    
    Method method1 = class_getInstanceMethod(cls, @selector(insertObject:atIndex:));

    Method method2 = class_getInstanceMethod(cls, @selector(lc_insertObject:atIndex:));

    method_exchangeImplementations(method1, method2);
}


-(void)lc_insertObject:(id)anObject atIndex:(NSUInteger)index{
        
    if (anObject == nil) {
        return;
    }
    
    [self lc_insertObject:anObject atIndex:index];
}

@end
```

<!-- ************************************************ -->
## <a id="content5"></a>利用消息转发机制，解决方法找不到的异常问题

Person.h
```objc
@interface Person : NSObject
-(void)test;
@end
```

Person.m文件
```objc
#import "Person.h"
#import <objc/runtime.h>


@implementation Person


-(void)extraFun{
    NSLog(@"lilog: %s",__func__);
}

- (NSMethodSignature *)methodSignatureForSelector:(SEL)aSelector{
    
    return [NSMethodSignature signatureWithObjCTypes:"v16@0:8"];
}

- (void)forwardInvocation:(NSInvocation *)anInvocation{
    
    NSLog(@"lilog: %s",__func__);
    anInvocation.selector = @selector(extraFun);
    [anInvocation invoke];
}
@end
```

调用
```objc
- (void)viewDidLoad{
    [super viewDidLoad];
    
    Person * person = [Person new];
    
    [person  test];
}
```



----------
>  行者常至，为者常成！


