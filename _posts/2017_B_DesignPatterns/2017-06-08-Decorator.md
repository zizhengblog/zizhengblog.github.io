---
layout: post
title: "8、Decorator"
date: 2017-06-08
description: ""
tag: 设计模式
---


- [参考：装饰器模式](https://www.runoob.com/design-pattern/decorator-pattern.html)





## 目录

* [装饰器模式](#content1)
* [代码实现](#content2)




<!-- ************************************************ -->
## <a id="content1"></a>装饰器模式

装饰器模式（Decorator Pattern）允许向一个现有的对象添加新的功能，同时又不改变其结构。这种类型的设计模式属于结构型模式，它是作为现有的类的一个包装。

<span style="color:red;font-weight:bold">这种模式创建了一个装饰类，用来包装原有的类，并在保持类方法签名完整性的前提下，提供了额外的功能。</span>

动态地给一个对象添加一些额外的职责。就增加功能来说，装饰器模式相比生成子类更为灵活。

一般的，我们为了扩展一个类经常使用继承方式实现，由于继承为类引入静态特征，并且随着扩展功能的增多，子类会很膨胀。

所以在不想增加很多子类的情况下扩展类,可以使用装饰器模式.

比如:

有一个圆形,有一个正方形.

我们想给圆形增加一个红色边框,可以继承圆形,写一个红色边框圆形

当我们想给正方形加一个红色边框,可以继承正方形,写一个红色边框正方形

某一天我们又想给圆形增加一个阴影,可以继承圆形,写一个阴影圆形

我们又想给圆形同时加上红色边框增加阴影....

这时类的设计会很复杂,我们可以使用装饰器模式来解决这个问题.

我们只要有一个红色边框装饰器,阴影装饰器,就可以解决:

红色边框装饰器(圆形),红色边框装饰器(正方形),阴影装饰器(圆形),红色边框装饰器(阴影装饰器(圆形))


<!-- ************************************************ -->
## <a id="content2"></a>代码实现

<img src="/images/DesignPatterns/decorator.png" alt="img">

**一 接口设计**

Shape.swift
```swift
protocol Shape {
    func draw()
}
```
**二 具体实现**

Circle.swift
```swift
class Circle: Shape {
    func draw() {
        print("draw a circle ...")
    }
}
```

Rectangle.swift
```swift
class Rectangle: Shape {
    func draw() {
        print("draw a rectangle ...")
    }
}
```

**三 装饰器接口设计**

```swift
protocol DecoratorShape: Shape {
    var shape:Shape{get set}
    init(shape:Shape)
}
```

**四 装饰器接口实现**

RedDecoratorShape.swift
```swift
class RedDecoratorShape: DecoratorShape {
    var shape: Shape
    
    required init(shape: Shape) {
        self.shape = shape
    }
    
    func draw() {
        shape.draw()
        setRedBorder()
    }
    
    private func setRedBorder() {
        //对shape添加红色边框
        print("set a red border ...")
    }
}
```


ShadowDecoratorShape.swift
```swift
class ShadowDecoratorShape: DecoratorShape {
    var shape: Shape

    required init(shape: Shape) {
        self.shape = shape
    }
    
    func draw() {
        shape.draw()
        setShadow()
    }
    
    private func setShadow() {
        //对shape添加阴影
        print("set a shadow ...")
    }
}
```

**五 调用**

```swift
func test1() {
    let circle = Circle()
    circle.draw()
    
    let circleDecorator = RedDecoratorShape(shape: circle)
    circleDecorator.draw()
    
    print("-------------------")
    
    let rect = Rectangle()
    rect.draw()
    let rectDecorator = RedDecoratorShape(shape: rect)
    rectDecorator.draw()
    
    let shadowDecorator = ShadowDecoratorShape(shape: rectDecorator);
    shadowDecorator.draw();
}
```

日志打印:

```
draw a circle ...
draw a circle ...
set a red border ...
-------------------
draw a rectangle ...
draw a rectangle ...
set a red border ...
draw a rectangle ...
set a red border ...
set a shadow ...
```



----------
>  行者常至，为者常成！


