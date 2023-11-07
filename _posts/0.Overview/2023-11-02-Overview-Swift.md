---
layout: post
title: "Swift概要"
date: 2023-11-02
tag: Overview
---





## 目录


- [常量](#content1)   
- [变量与数据类型](#content2)   
- [闭包](#content3)   
- [可选项](#content4)   
- [枚举/结构体/类](#content5)   
- [协议](#content6)   
- [泛型](#content7)   
- [其它](#content10)   



Swift中的重要概念


<!--===============================================================================================-->
## <a id="content1">常量</a>
```Swift
//不要求编译时确定，如果声明时没有值需要指定类型
let a: Int = 10 

//可以省略类型说明和结尾的分号
let a = 10 
```

<!--===============================================================================================-->
## <a id="content2">变量与数据类型</a>

#### 一、变量
```Swift
var isClick:Bool = false //可以省略类型说明和结尾的分号

var a:Int = 0; 
var a:Float = 10.0; 
var a:Double = 10.0;

var cha:Character = "a" // 字符类型必须指定类型
var str:String = "hello world"

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

#### 二、数据类型

##### 1、String
swift中的字符串类型是结构体，是值类型<br>
值类型的特点是：将a赋值给b之后，对b进行修改不会影响a

```swift
//1、长度小于等于15的字符串存储于str的内存中
var str:String = "0123456789"

//2、append后长度仍小于等于15的字符串 仍位于变量的内存
str.append("A")

//3、长度大于15的字符串位于常量区（常量区的内存在编译时分配，程序运行时无法更改）
var str = "0123456789ABCDEF"

//4、不管之前长度是否大于15，append后长度大于15的字符串 会存储于堆空间
str.append("ABCDEF")

```

##### 2、Array
Swift中的数组也是结构体，是值类型
```swift
var arr:Array = [1,2,3,4]

var arr1 = arr
arr1.append(6)

//arr1的改变并不会影响arr
//arr1 = [1, 2, 3, 4, 6], arr = [1, 2, 3, 4]
print("arr1 = \(arr1), arr = \(arr)")
```
<span style="color:red;fontWeight:bold">PS:Array在底层其实是引用类型 但实际使用时是结构体<br> 行为上是值类型 本质是引用类型只是苹果隐藏了这一层</span>

##### 3、Dictionary 和 Set 也是结构体类型，也是值类型

##### 4、元组是一个复合类型，也是值类型
 
<!--===============================================================================================-->
## <a id="content3">闭包</a>

什么是闭包？<br>
    外层函数 + 内层函数 + 内层函数访问外层函数的变量<br>

闭包的作用？<br>
    1、捕获变量，在函数调用结束后仍能访问函数内部的变量：比如计数器<br>
    2、代码模块化，比如返回一个包含了多个方法的元组或对象：比如计算器<br>
    3、异步任务，函数调用结束后还能执行捕获的闭包表达式：网络请求<br>

逃逸闭包？<br>
    非逃逸闭包：内部定义，内部调用<br>
    逃逸闭包：内部定义，外部调用<br>
  
 
<!--===============================================================================================-->
## <a id="content4">可选项</a>
#### 一、空判断和强制解包
```Swift
let a:Int? = 10

if a! = nil {
    let num = a!
}
```

#### 二、自动解包

##### 1、可选项绑定自动解包
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

##### 2、! 自动解包
```Swift
let a:Int? = 1

let b:Int! = 2

// b可以赋值给c，a不行
let c:Int = b
```


#### 三、空合并运算符
```Swift
let a:Int? = 1

let b:Int? = 2

// a不为nil返回a,a为nil返回b.
// 返回的类型与b相同
let c = a ?? b
```



#### 四、多重可选项
```
多重可选项的比较：
    能解包出相同的具体值的多重可选项都是相等的

使用场景：GPT异步比如获取用户信息。没明白??
```

#### 五、可选链
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
//num是nil会终止后续操作，不会赋值10
num? = 10
```

#### 六、可选项的本质是枚举

##### 1、举例
```swift
//可选项
var num:Int? = nil
print(num)  // nil
num = 10
print(num)  // Optional(10)

// 枚举
var num1:Optional<Int> = nil
print(num1) // nil
num1 = 10
print(num1) // Optional(10)


// .none 就是nil
var num2:Optional<Int> = .none
print(num2) // nil
num2 = .some(10)
print(num2) // Optional(10)
```

##### 2、自己实现
```swift
enum MYOptional<T> : ExpressibleByNilLiteral & ExpressibleByIntegerLiteral & CustomStringConvertible{
    case none
    case some(T)
    
    // 字面量协议： ExpressibleByNilLiteral
    init(nilLiteral: ()) {
        self = .none
    }
    
    // 字面量协议：ExpressibleByIntegerLiteral
    init(integerLiteral : Int) {
        self = .some(integerLiteral as! T)
    }

    // CustomStringConvertible 协议
    var description: String {
        switch self {
        case .none:
            return "nil"
        case let .some(val):
            return "Optional(\(val))"
        }
    }
}


var num3:MYOptional<Int> =  nil
print(num3) // nil
num3 = 10
print(num3) // Optional(10)


var num4:MYOptional<Int> = .none
print(num4) // nil
num4 = .some(10)
print(num4) // Optional(10)
```


<!--===============================================================================================-->
## <a id="content5">枚举/结构体/类</a>

### 枚举

```
原始值 内存占用1字节
关联值 内存占用 1字节 + n字节
枚举内也可以定义方法
```

### 结构体和类
结构体是值类型  内存分布：栈空间<br>
let 对结构体变量和类变量的不同，理解let的本质


### 类

#### 基本结构

类是引用类型   内存分布：堆空间(16的倍数) 前8字节isa 再8字节引用计数。let p = Person() p占用8个字节<br>
嵌套的类型不占用外部类型的空间

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


#### 实例对象与类对象
```Swift
class Person {}
let p :Person = Person()
let cls :Person.type = Person.self
// p 对应OC中的实例对象
// Person.self 对应OC中的类对象
// Person.type 对应OC中的Class
// Person 与 Person.self 都能调用类方法
// let pcls : Person.type = Person.self; 但是不能 let pcls:Person.type = Person
```


<!--===============================================================================================-->
## <a id="content6">协议</a>

#### 一、一个完整的协议格式
```Swift
protocol Life {
    //Property in protocol must have explicit { get } or { get set } specifier
    var age:Int{get}
    var name:String{get set}
    init(age:Int, name:String)//也可以int?(xxx) int!(xxx)
    func run()
    mutating func grow()
    static func walk()
}

//协议的继承
protocol Dog : Life {
    func fake()
}

protocol Person : Life {
    var car:String{get}
}

protocol Teach {
    var studens:Array<Any>{get}
}


//协议的使用
class Teacher : Person & Teach {
    xxx
}
```

#### 二、带关联类型的协议
```swift
protocol Runable {
    associatedtype Speed // 关联类型
    func run(s:Speed)
}

//带关联类型和Self的协议只能作为泛型的约束，不能作为参数类型和返回值类型
//xy:参数类型和返回值类型需要确定的类型，不能包含不透明类型(协议内的关联类型为不透明类型或者叫不确定类型)
//要想带关联类型和Self的协议作为参数类型和返回值类型可以使用some关键字
func setObj(obj:some Runable) {
    
}
```

#### 三、其它

```
inout的本质：地址传递
// 计算属性 和 设置了属性观察器的属性
copy in copy out(会产生一个副本)
```

```Swift
单利对象
public static let share = FileManager()
private func init(){}
```

<!--===============================================================================================-->
## <a id="content7">泛型</a>
#### 一、泛型就是将类型作为参数，或者说是类型的占位

```swift
// 方法与泛型
func test<T>(num:T) -> T {
    //返回值的类型也应该是T
    // 不能返回 num + 10,因为不知道num是否支持+号运算，也不知道(num+10)是否是T类型
    // 泛型在使用时要将其作为一个特定类型使用它就是T类型而不是其它任何类型。
    return num
}
test(num:10)

//枚举与泛型
enum Score<T>{
    case point(T)
    case grade(String)
}
let Score<Int>.point(10)


//类与泛型
class Stack<T> {
    var elements = [T]()
    func push(element:T) {
        elements.append(element)
    }
}
let st = Stack<Int>()
st.push(element:10)


//类与泛型与协议
protocol Stackable {
    associatedtype E // 关联类型
    mutating func push(element: E)
}

class Stack<T> : Stackable {
    //指定协议内的关联类型为具体类型
    //typealias E = Int
    //func push(element:E) {}
    
    //指定关联类型为泛型
    //typealias E = T
    //func push(element:E) {}
    
    //自定指定关联类型为Int
    //func push(element: Int) {}
    
    // 自定指定关联类型为T
    func push(element: T) {}
    
    // typealias 就是一个别名
}
```

#### 二、可以给泛型指定约束
```swift

// 用协议约束泛型
func compaire<T : Equatable>(num1:T, num2:T) -> Bool {
    return num1 == num2
}

// 用带有关联类型的协议约束泛型
protocol Runable {
    associatedtype Speed // 关联类型
    func run(s:Speed)
}
func setObj<T:Runable>(obj:T) {
}

class Animal : Runable {
    //指定关联类型为Int
    func run(s: Int) {}
}

let ani =  Animal()
setObj(obj:ani)
```
#### 三、some关键字

带关联类型和Self的协议只能作为泛型的约束，不能作为参数类型和返回值类型<br>
xy:参数类型和返回值类型需要确定的类型，不能包含不透明类型(协议内的关联类型为不透明类型或者叫不确定类型)<br>
```swift
// 下面两个方法会报错

//Protocol 'Runnable4' can only be used as a generic constraint because it has Self or associated type requirements
func get() -> Runable{
    return Animal() as! Runable
}

//Protocol 'Runnable4' can only be used as a generic constraint because it has Self or associated type requirements
func setobj(obj:Runable) {

}
```
要想带关联类型和Self的协议作为参数类型和返回值类型可以使用some关键字<br>
```swift
func get() -> some Runable{
    return Animal() as! Runable
}

func setobj(obj:some Runable) {

}
```

还可以使用泛型解决
```swift
func get<T:Runable> () -> T {

}

func set<T:Runable>(obj:T) {
    
}
```



















<!--===============================================================================================-->
## <a id="content10">其它</a>

mutating的使用<br>
subscript的使用<br>
final的使用：禁止被子类重写和继承<br>


进度：14-01


----------
>  行者常至，为者常成！


