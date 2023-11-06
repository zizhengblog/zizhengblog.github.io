---
layout: post
title: "Language"
date: 2023-11-05
tag: Overview
---





## 目录


- [常量](#content1)   
- [变量与数据类型](#content2)   
- [分支结构](#content3)   
- [循环结构](#content4)   



```
代码编译之后是二进制文件也就是0101指令
计算机执行二进制文件就是运行指令:开辟空间+处理操作数
静态语言：编译时检查类型
动态语言：运行时检查类型
```


<!-- ************************************************ -->
## <a id="content1">常量</a>
```
Object-C
    const int a = 10;

Swift
    let a: Int = 10 //不要求编译时确定，如果声明时没有值需要指定类型
    let a = 10 //可以省略类型说明和结尾的分号

JavaScript
    const a = 10

Flutter
    const int a = 10;   //编译时确定
    const a = 10;       //int 可以省略
    finial int a = 10;  //运行时确定
```


<!-- ************************************************ -->
## <a id="content2">变量与数据类型</a>
```
Object-C
    BOOL isClick = NO; 
    int a = 0; float a = 10.0;
    NSString *str = @"hello world";
    NSArray *array = @[@(1), @(2), @(3)];
    NSDictionary *dic = @{@"key1":@"valu1", @"key2": @"value2"}
    XYPerson *person = [[XYPerson alloc] initWithName:@"xiaoming" age:18];
    // 匿名函数
    int(^blk)(int a, NSString* str) = ^int(int a, NSString* str){  return 10;};
    // 函数
    -(int)sumWithNumA:(int)numA numB:(int)numB;

Swift
    var isClick:Bool = false //可以省略类型说明和结尾的分号
    var a:Int = 0; var a:Float = 10.0; a:Double = 10.0;
    var str:String = "hello world"
    var cha:Character = "a" // 字符类型必须指定类型
    var array:Array = [1, 2, 3, 4]
    var dic : Dictionary = ["key1":"value1", 1:"value2"]
    var set:Set = ["age","weight"]
    // 元组
    let error = (404, "Not Fount")// error.0
    let error = (code:404, msg:"Not Fount")//error.code
    //对象类型
    let p:Person = Person(name:"xiaoming",age:18)
    //匿名函数:闭包表达式，可以作为参数，尾随闭包的调用
    var blk:(Int,Int)->Int = {(a, b) in return a + b}   // var result = blk(10,20)
    //函数
    func sum(a:Int, b:Int)->Int {return 10}
    func goToWork(at time:String){} // goToWork(at:"10:00")


JavaScript
    let isClick = false
    let a = 0
    let str = 'hello world'
    let array = [1, 2, 3]
    let dic = {key1:'value1', key2:'value2'}
    let person = new Person('xiaoming', 18)
    undefine
    // 匿名函数
    let blk = (a, str)=>{ return 10 }
    let blk = function(a, str){ return 10}
    // 函数
    function sum (numA, numB) {}

Flutter
    bool isClick = false;
    var isClick = false;    //可以使用var 自动推断类型
    int a = 10; double a = 10.0; num a = 10;
    String str = 'hello world';
    List array = [1, 2, 3]
    Map dic = {'key1': 'hank', 'key2': 10};
    Person person = Person('xiaoming', 18);
    // 匿名函数
    var blk = (a, str){return 10;}
    // 函数:参数类型和返回值类型可以省略
    int sum (numA, numB){}
```


<!-- ************************************************ -->
## <a id="content3">分支结构</a>

```
Object-C
    int a = 10
    if ( a > 10){

    } else if (a > 5) {

    } else {

    }

    switch (a) {
        case 10:
            break;
        case 5:
            break;
        default:   
    }
Swift
    var a = 10
    if a > 10 {

    } else if a > 5 {

    } else {

    }

    switch a {
        case 10 :

        case 5 :

        default:

    }

JavaScript
    let a = 10
    if ( a > 10){

    } else if (a > 5) {

    } else {

    }

    switch (a) {
        case 10:
            break;
        case 5:
            break;
        default:    
    }

Flutter
    int a = 10
    if ( a > 10){

    } else if (a > 5) {

    } else {

    }

    switch (a) {
        case 10:
            break;
        case 5:
            break;
        default:    
    }

```

<!-- ************************************************ -->
## <a id="content3">循环结构</a>

```
    Object-C
        for (NSNumber *num in arary) {}
        for (int i = 0; i < length; i++){}
        while(YES){}

    Swift 
        for i in 1...3 {}
        for i in array {}
        for i in 1..<array.count {}
        while true {}

    JavaScript
        for (let key in array) {}
        for (let i = 0; i < length; i++){}
        while (true) {}

    Flutter
        for (int num in arary) {}
        for (int i = 0; i < length; i++){}
        while(YES){}


五、面向过程和面向对象
    面向过程
        JavaScript


    面向对象:封装、继承、多肽
        Object-C
            构造方法:
                -(void)init;    //构造方法
                -(void)dealloc; //析构方法
            实例属性:
                @property (nonatomic, assign) NSString* name;
            实例方法:
                -(int)sumWithNumA:(int)numA numB:(int)numB;
            静态属性:
                无
            静态方法:
                +(int)sumWithNumA:(int)numA numB:(int)numB;
            继承:
                @interface Animal : NSObjec
                @end

                @interface Dog : Animal
                @end
                [super init];

        Flutter
            构造方法:
                Person(this.name, this.age) : sexy = true, assert(age >= 0);   // 后边跟着初始化列表 和 断言
                Person.all(this.name, this.age, this._height, this._hobby) : sexy = true; //命名构造方法

            实例属性:
                String name;
                int _height;
            计算属性:
                float get whithVsHeight { return 0.5;}  // 没有参数列表 本质是方法
            实例方法:
                int sum(numA, numB){ return 10;}
                int _sum(numA, numB){ return 10;}

            静态属性：
                static int? num;
            静态方法：
                static int sum(numA, numB){ return 10;}

            继承:
                class Animal {}
                //abstract class Animal{}   // 抽象类：接口/协议
                class Dog extends Animal {}
```

            
            

----------
>  行者常至，为者常成！


