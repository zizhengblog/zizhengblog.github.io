---
layout: post
title: "JavaScriptCore"
date: 2018-03-11
tag: Objective-C
--- 
- [美团技术团队：深入理解JSCore](https://tech.meituan.com/2018/08/23/deep-understanding-of-jscore.html)
- [重要：深入浅出 JavaScriptCore](https://www.jianshu.com/p/ac534f508fb0)
- [重要：上面文章中在 Github 上的Demo](https://github.com/DengKaiHui/Demo_JSExport)





## 目录
* [iOS的JavaScriptCore框架](#content1)
* [JSExport协议](#content2)
* [Demo](#content3)



<!-- ************************************************ -->
## <a id="content1">iOS的JavaScriptCore框架</a>

JavaScriptCore是苹果公司提供的JavaScript引擎，用于在iOS和macOS上**执行JavaScript代码**。      
它是WebKit框架的一部分，WebKit是苹果用于处理Web内容的开源引擎。    
JavaScriptCore提供了与JavaScript交互的接口，使开发者能够在原生应用中使用JavaScript来实现一些功能。

iOS中可以使用JSCore的地方有多处，比如封装在UIWebView中的JSCore，封装在WKWebView中的JSCore，以及系统提供的JSCore。

很有必要了解的概念只有4个：JSVM，JSContext，JSValue，JSExport。

**JSVirtualMachine**    
一个JSVirtualMachine（以下简称JSVM）实例代表了一个自包含的JS运行环境，或者是一系列JS运行所需的资源。    
该类有两个主要的使用用途：一是支持并发的JS调用，二是管理JS和Native之间桥对象的内存。      
   
**JSContext**     
是一个关键的类，它代表了JavaScript的执行环境。

**JSValue**    
可以表示JavaScript中的各种基本类型，包括数字、字符串、布尔值、对象、数组和函数等。是类型转换的桥梁。              

**JSExport**   
通过采用JSExport协议，你可以将Objective-C对象的方法和属性暴露给JavaScript环境，使得它们在JavaScript代码中可以直接调用。  

<img src="/images/objectC/objc18.png">


#### **一、代码演示**    

**1、执行一段js代码**               
```objc
JSContext *context = [[JSContext alloc] init];

// 执行一段js代码，并获取返回的结果，JSValue代表js对象
JSValue *result = [context evaluateScript:@"2 + 2"];

// JS对象转为OC对象
NSLog(@"Result: %@", [result toString]);
```

**2、从js环境中设置和获取属性(字符串)**       
```objc
JSContext *context = [[JSContext alloc] init];

// 向js环境中设置一个字符串
[context setObject:@"Hello from Objective-C!" forKeyedSubscript:@"message"];

//或者这种方式
context[@"message"] = @"Hello from Objective-C!";


// 从js环境中获取一个字符串
JSValue *message = context[@"message"];
NSLog(@"Message from JavaScript: %@", [message toString]);
```

**3、从js环境中设置和获取属性(对象)**       
```objc
JSContext *context = [[JSContext alloc] init];

// 向js环境中设置一个对象
JSValue *dataValue = [JSValue valueWithObject:@{@"key": @"value"} inContext:context];
[context setObject:dataValue forKeyedSubscript:@"myData"];

// 从js环境中获取并修改对象
JSValue *myVarValue = context[@"myVar"];
[myVarValue setValue:@42 forKeyedSubscript:@"value"];
```

**4、从js环境中设置和获取方法**         
```objc
JSContext *context = [[JSContext alloc] init];

// 从js环境中获取一个方法并调用
JSValue *function = context[@"myFunction"];
JSValue *result = [function callWithArguments:@[@"arg1", @"arg2"]];
NSLog(@"Result from JavaScript function: %@", [result toString]);


// 向js环境中添加一个方法：myObjectiveCMethod，这个方法js可以调用myObjectiveCMethod()
context[@"myObjectiveCMethod"] = ^() {
    NSLog(@"Objective-C method called from JavaScript");
};
``` 

通过 `context[key] = value`这种方式添加到js环境中的属性，都会被添加到一个全局的对象下边，可以通过下面这种方式查看     
```objc
context.globalObject.toDictionary
```

#### **二、js 和 native 的相互调用**      
```objc
JSContext *context = [[JSContext alloc] init];
// 在Objective-C中调用JavaScript函数
// 调用定义在js环境中方法：myJavaScriptFunction
[context evaluateScript:@"myJavaScriptFunction();"];

// 从JavaScript中调用Objective-C方法
// 向js环境中添加一个方法：myObjectiveCMethod
context[@"myObjectiveCMethod"] = ^() {
    NSLog(@"Objective-C method called from JavaScript");
};
```


<!-- ************************************************ -->
## <a id="content2">JSExport协议</a>

通过采用JSExport协议，你可以将Objective-C对象的方法和属性暴露给JavaScript环境，使得它们在JavaScript代码中可以直接调用。    

**1、定义在Objective-C中的接口：**      
在Objective-C中，你定义一个遵循JSExport协议的协议，并在其中声明你想要在JavaScript中使用的方法和属性。

```objc
#import <JavaScriptCore/JavaScriptCore.h>

@protocol XYJSPointExports <JSExport, NSObject>

@property (nonatomic, assign) double x;
@property (nonatomic, assign) double y;

- (NSString *)description;

+ (XYJSPoint *)makePointWithX:(double)x Y:(double)y;

//JSExportAs(makePoint,
//+ (MyPoint *)makePointWithX:(double)x Y:(double)y
//);
@end
```

**2、在Objective-C类中实现JSExport协议：**     
在Objective-C中的类中，实现上述定义的MyJSExport协议。    

```objc
@interface XYJSPoint : NSObject <XYJSPointExports>

- (instancetype)initWithX:(double)x Y:(double)y;

@end
```

```objc
@implementation XYJSPoint
@synthesize x, y;
- (instancetype)initWithX:(double)x
                        Y:(double)y {
    self = [super init];
    if (self) {
        self.x = x;
        self.y = y;
    }
    return self;
}

- (NSString *)description {
    NSString *str = [NSString stringWithFormat:@"(%f,%f)",self.x,self.y];
    return str;
}

+ (id)makePointWithX:(double)x Y:(double)y {
    XYJSPoint *point = [[XYJSPoint alloc] initWithX:x Y:y];
    return point;
}

@end
```

**3、关联JSContext：**      
将MyObject类的实例关联到JSContext，以便在JavaScript中可以访问它的属性和方法。    
```objc
JSContext *context = [[JSContext alloc] init];
XYJSPoint *point = [[MyObject alloc] init];
context[@"point"] = point;
```

**4、在JavaScript中使用：**   
在JavaScript中，你可以直接访问myObject的属性和方法，就像在JavaScript中调用普通的对象一样。    

```js
point.x = 3;
point.description(); 
```
通过使用JSExport协议，你可以轻松地在Objective-C和JavaScript之间定义和传递接口，使得两种语言之间的交互更加方便。    
这对于在iOS应用中嵌入Web内容，或者在JavaScript中使用Objective-C实现的功能非常有用。     

**5、关系图如下**   

<img src="/images/objectC/objc19.webp">

oc对象属性 - js对象的原型对象中属性      
oc对象方法 - js对象的原型对象中方法      
oc类方法 - js对象构造函数的静态方法     
oc的继承关系 - js的原型链继承     


**6、内存管理**      
不要在JS中给OC对象增加成员变量    
OC对象不要直接强引用JSValue对象(有解决方法，查阅参考文档)        

<!-- ************************************************ -->
## <a id="content3">Demo</a>

**1、factorial.js内的代码如下：**        
```js
// 计算阶乘
var factorial = function (n) {
    if (n < 0)
        return;
    if (n === 0)
        return 1;
    return n * factorial(n-1)
};


// 计算两点间的距离
var euclideanDistance = function(p1, p2) {
    var xDelta = p2.x - p1.x;
    var yDelta = p2.y - p1.y;
    return Math.sqrt(xDelta * xDelta + yDelta * yDelta);
};


// 计算中点
var midpoint = function(p1, p2) {
    var xDelta = (p2.x - p1.x) / 2;
    var yDelta = (p2.y - p1.y) / 2;
    return XYJSPoint.makePointWithXY(p1.x + xDelta, p1.y + yDelta);
};
```

**2、加载js代码**  
```objc
- (NSString *)loadJSFromBundle:(NSString *)jsfileName {
    NSString * path = [[NSBundle xy_bundleName:@"XYTestModule"] pathForResource:jsfileName ofType:nil];
    NSString *jsStr = [NSString stringWithContentsOfFile:path encoding:NSUTF8StringEncoding error:nil];
    return jsStr;
}
```

**3、Demo1**  
```objc
// oc调用js代码中的方法
- (IBAction)jscoreTest1:(id)sender {
    JSContext *context = [[JSContext alloc] init];
    NSString *jsStr = [self loadJSFromBundle:@"factorial.js"];
    // 加载js代码
    [context evaluateScript:jsStr];
    
    JSValue *func = context[@"factorial"];
    JSValue * result = [func callWithArguments:@[@4]];
    
    NSLog(@"lxy:result = %d",[result toInt32]);
}
```

**3、Demo2**  

```objc
//JSExport协议的使用
- (IBAction)jscoreTest3:(id)sender {
    
    // 加载js代码
    JSContext *context = [[JSContext alloc] init];
    NSString *jsStr = [self loadJSFromBundle:@"factorial.js"];
    [context evaluateScript:jsStr];
    

    // 创建2个点
    XYJSPoint *point1 = [[XYJSPoint alloc] initWithX:0.0 Y:0.0];
    XYJSPoint *point2 = [[XYJSPoint alloc] initWithX:3.0 Y:4.0];
    
    // 调用JS方法，求得两点间的距离
    JSValue *function = context[@"euclideanDistance"];
    JSValue *result = [function callWithArguments:@[point1, point2]];
    NSLog(@"result = %f",[result toDouble]);
    
    // 调用JS方法，求得两点间的中点
    context[@"XYJSPoint"] = [XYJSPoint class];

    JSValue *function2 = context[@"midpoint"];
    JSValue *jsResult = [function2 callWithArguments:@[point1, point2]];
    XYJSPoint *midpoint = [jsResult toObject];
    NSLog(@"midpoint = %@",midpoint);
}
```


----------
>  行者常至，为者常成！



