---
layout: post
title: "Swift概要"
date: 2023-11-02
tag: Overview
---





## 目录


- [常量](#content1)   
- [变量与数据类型](#content2)   
- [可选项](#content3)   
- [结构体和类](#content4)   
- [闭包](#content5)   
- [其它](#content6)   




Swift中的重要概念


<!-- ************************************************ -->
## <a id="content1">常量</a>
```Swift
//不要求编译时确定，如果声明时没有值需要指定类型
let a: Int = 10 

//可以省略类型说明和结尾的分号
let a = 10 
```

<!-- ************************************************ -->
## <a id="content2">变量与数据类型</a>

```Swift
var isClick:Bool = false //可以省略类型说明和结尾的分号

var a:Int = 0; 
var a:Float = 10.0; 
var a:Double = 10.0;

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
```
 
 
 
<!-- ************************************************ -->
## <a id="content3">可选项</a>
空判断和强制解包
```Swift
let a:Int? = 10

if a! = nil {
    let num = a!
}
```

可选项绑定：自动解包
```Swift
// num的作用域是块作用域
if let num=a, num>0 {

}

while let num=a, num>0 {

}

// userName的作用域是函数作用域
// 字典返回的是可选项
guard let userName = info["userName"] { 
    return 
}
print(userName)
```


空合并运算符
```Swift
let a:Int? = 1

let b:Int? = 2

let c = a ?? b
```

自动解包
```Swift
let a:Int? = 1

let b:Int! = 2

let c:Int = b
```

多重可选项
```
使用场景：GPT异步比如获取用户信息。没明白??
多重可选项的比较
```

可选链
```Swift
let person:Person? = Person()
//如果不是可选类型会包装成可选类型
let age = person?.age // optional
let age = person!.age // int

// 如果是可选类型不会重复包装
// car 是可选类型
let car = person?.car //optional
let car = person!.car //optional


// 如果person是nil就会终止后续操作
person?.run()

let num:Int? = 5
let num:Int? = nil
num? = 10

```


<!-- ************************************************ -->
## <a id="content4">结构体和类</a>

枚举
```
原始值 内存占用1字节
关联值 内存占用 1字节 + n字节
枚举内也可以定义方法
```

结构体和类
```
结构体是值类型  内存分布：栈空间
类是引用类型   内存分布：堆空间(16的倍数) 前8字节isa 再8字节引用计数。let p = Person() p占用8个字节

let 对结构体变量和类变量的不同，理解let的本质

嵌套的类型不占用外部类型的空间
```


方法和属性
```
实例属性：
    存储属性
    计算属性
实例方法：
    init方法(重要)：指定初始化器 + 便捷初始化器，以及父子类中有哪些规则
    deinit方法：先子类后父类


静态属性：
    存储属性
    计算属性
静态方法
```


inout的本质
```
地址传递
copy in copy out(会产生一个副本)
```

单利对象
```Swift
public static let share = FileManager()

private func init(){

}
```



<!-- ************************************************ -->
## <a id="content5">闭包</a>

什么是闭包？<br>
    外层函数 + 内层函数 + 内层函数访问外层函数的变量<br>

闭包的作用？<br>
    1、捕获变量，在函数调用结束后仍能访问函数内部的变量：比如计数器<br>
    2、代码模块化，比如返回一个包含了多个方法的元组或对象：比如计算器<br>
    3、异步任务，函数调用结束后还能执行捕获的闭包表达式：网络请求<br>

逃逸闭包？<br>
    非逃逸闭包：内部定义，内部调用<br>
    逃逸闭包：内部定义，外部调用<br>
 


<!-- ************************************************ -->
## <a id="content6">其它</a>

mutating的使用<br>
subscript的使用<br>
final的使用：禁止被子类重写和继承<br>


进度：11-01 other11


----------
>  行者常至，为者常成！


