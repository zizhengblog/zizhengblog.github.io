---
layout: post
title: "JavaScript概要"
date: 2023-09-02
tag: Overview
---   




## 目录
* [语言](#content1)
* [变量/函数提升](#content2)
* [构造函数](#content3)
* [原型对象](#content4)







<!-- ************************************************ -->
## <a id="content1">语言</a>

#### **一、常量变量**  
```js
// 常量：不允许修改
const pi = 3.14 

// 变量：可以修改
let num = 99 
num= 100 
```

#### **二、基本数据类型**   
```js
// boolean
let isClick = false

// number(NaN也是number类型 浮点也是number类型)
let a = 0

// string
let str = 'hello world'

// undefine是一种类型
undefine
```

#### **三、对象类型**  
```js
let array = [1, 2, 3]

let dic = {key1:'value1', key2:'value2'}

let person = new Person('xiaoming', 18)

// 匿名函数
let blk = (a, str)=>{ return 10 }
let blk = function(a, str){ return 10}

// 函数
function sum (numA, numB) {}

// null 也是object类型
let test = null
```

#### **四、一门语言有哪些基本要素**     

**1、分支结构、循环结构**  

**2、面向对象、面向过程**    
封装、继承、多肽      
属性      
方法：构造函数、析构函数、普通函数    

**3、多线程、同步异步**   

**4、其它**   
运算：四则/逻辑/位运算    
事件：捕获与冒泡    
定时器：间隔执行、到点执行    


<!-- ************************************************ -->
## <a id="content2">变量/函数提升</a>

#### **一、变量提升**  

当你使用 `var` 关键字声明变量时，这个变量的声明会被提升到当前函数或全局作用域的顶部

```js
// 输出: undefined
console.log(x); 
var x = 10;

// 相当于
var x;
console.log(x); 
x = 10;
```

理解：函数作用域 和 块级作用域{}     
```js
// 使用 let 声明的变量，具有块级作用域
if (true) {
    // var blockScopedVar = 'I am block scoped';
    let blockScopedVar = 'I am block scoped';
}
console.log(blockScopedVar); // 报错：blockScopedVar is not defined


for (var i = 0; i < 5; i++) {
    setTimeout(function () {
        console.log(i); // 输出: 5，5，5，5，5（因为 i 具有函数作用域，共享相同的 i 变量）
    }, 1000);
}

for (let j = 0; j < 5; j++) {
    setTimeout(function () {
        console.log(j); // 输出: 0，1，2，3，4（因为 j 具有块级作用域，每个循环迭代都有自己的 j 变量）
    }, 1000);
}
```

#### **二、函数提升**    
声明一个函数时，整个函数体（包括函数名称和函数体内的内容）会被提升到当前函数或全局作用域的顶部    

```js
greet(); // 输出: Hello!
function greet() {
    console.log('Hello!');
}


// 实际上，上述代码在执行时会被解释为以下形式：
function greet() {
    console.log('Hello!');
}
greet();
```


<!-- ************************************************ -->
## <a id="content3">构造函数</a>

#### **一、构造函数**         
1、创建实例对象(属性 + 方法)     
2、方法内this指向实例对象   
3、person1.sing 和 person2.sing并不是同一个方法

```js
/*
命名以大写字母开头
只能由 "new" 操作符来执行 - 实例化
构造函数返回实例化对象，return 返回的值无效，所以不要写return
构造函数内部的this指向实例化对象
*/
function Person(name = '', age = 1) {
    this.name = name
    this.age = age
    this.sing = () => {
        console.log('我会唱歌')
    }
    this.dance = function () {
        console.log('我会跳舞')
    }
}

// 创建实例对象：因为参数有默认值，可以不传参
let person1 = new Person()
let person2 = new Person('qiaozhi', 6)

//获取属性、调用方法
console.log(`person = ${person.name}-${person.age}`)
person.sing()
person.dance()

//false
console.log(person1.sing == person2.sing);
```

#### **二、静态属性/方法**      
```js
/*
构造函数的属性和方法被称为静态成员
静态属性，静态方法
静态成员方法中的 this 指向构造函数本身
*/
Person.eyes = 2
Person.walk = function () {
    console.log('人多会走路', this.eyes)
}
console.log(Person.eyes, Person.walk());
```


<!-- ************************************************ -->
## <a id="content4">原型对象</a>

原型对象的本质，它也是一个对象     
共享方法   
共享属性   
可以节省内存   

#### **一、原型对象**   

**原型对象共享方法**    
```js
// 在Person构造函数的原型对象上添加一个方法
Person.prototype.sing = function () {
    //构造函数和原型对象中的this 都指向 实例化的对象
    console.log(`${this.name}正在唱歌...`);
}

// 创建两个Person对象
let ldh = new Person('刘德华', 50)
let zxy = new Person('张学友', 55)

// 这两个对象共享相同的原型对象上的sing方法
ldh.sing()// 输出:刘德华正在唱歌
zxy.sing()// 输出:张学友正在唱歌

// 这是同一个方法
console.log(ldh.sing === zxy.sing); // 输出：true
```

**原型对象共享属性**    

```js
// 定义一个共享的age属性并初始化为0
Person.prototype.age = 0; 
const person1 = new Person('Alice');
const person2 = new Person('Bob');
console.log(person1.age); // 输出: 0
console.log(person2.age); // 输出: 0


// 修改了person1的age属性.
//实际上创建了一个名为age的属性并将其分配给person1对象，而不会修改原型对象上的age属性
person1.age = 25;
console.log(person1.age); // 输出: 25
console.log(person2.age); // 输出: 0（person2的age属性仍然是0，因为它没有被修改）
```














----------
>  行者常至，为者常成！



