---
layout: post
title: "4、AbstractFactory"
date: 2014-06-04
description: ""
tag: 设计模式
---






## 目录

* [抽象工厂模式](#content1)
* [抽象工厂实现 - 工厂类](#content2)
* [抽象工厂实现 - 产品类](#content3)
* [抽象工厂实现 - 使用](#content4)
* [总结](#content5)







<!-- ************************************************ -->
## <a id="content1"></a>抽象工厂模式

简单工厂模式和工厂模式，解决了增加产品对象时的可扩展性，但都是增加的单一类产品，比如CardWatchCenter和Vehicle接口。

当我们需要增加产品族（比如冰箱、电视、洗衣机三个物品就是一个产品族）时，就可以使用抽象工厂

<img src="/images/DesignPatterns/abstractFactory.png" alt="img">



<!-- ************************************************ -->
## <a id="content2"></a>抽象工厂实现 - 工厂类

**一、创建抽象工厂接口**
```swift
protocol Factory {
    func createTelevison() -> Televison
    func createFridge() -> Fridge
    func createWashingMachine() -> WashingMachine
}
```

**二、创建实现接口的实体类**

1、SamsungFactory.swift

```swift
class SamsungFactory: Factory {
    
    func createTelevison() -> Televison {
        //制作过程
        
        return SamsungTelevison()
    }
    
    func createFridge() -> Fridge {
        //制作过程
        
        return SamsungFridge()
    }
    
    func createWashingMachine() -> WashingMachine {
        //制作过程
        
        return SamsungWashingMachine()
    }

}
```

2、HaierFactory.swift

```swift
class HaierFactory: Factory {
    
    func createTelevison() -> Televison {
        //制作过程
        
        return HaierTelevison()
    }
    
    
    func createFridge() -> Fridge {
        //制作过程
        
        return HaierFridge()
    }
    
    func createWashingMachine() -> WashingMachine {
        //制作过程
        
        return HaierWashingMachine()
    }
}
```

<!-- ************************************************ -->
## <a id="content3"></a>抽象工厂实现 - 产品类

**一、创建抽象产品接口**

1、Televison.swift

```swift
protocol Televison {
    func show()
}
```

2、Fridge.swift

```
protocol Fridge {
    func cool() 
}
```

3、washingMachine.swift

```
protocol WashingMachine {
    func wash()
}
```

**二、创建实现接口的实体类**

1、SamsungTelevison.swift

```swift
class SamsungTelevison: Televison {
    func show() {
        print("Samsung televison showing ...")
    }
}
```

2、HaierTelevison.swift

```swift
class HaierTelevison: Televison {
    func show() {
        print("Haier televison showing ...")
    }
}
```

3、SamsungFridge.swift

```swift
class SamsungFridge: Fridge {
    func cool() {
        print("Samsung fridge cooling ...")
    }
}
```

4、HaierFridge.swift

```swift
class HaierFridge: Fridge {
    func cool() {
        print("Haier Fridge cooling ...")
    }
}
```

5、SamsungWashingMachine.swift

```swift
class SamsungWashingMachine: WashingMachine {
    func wash() {
        print("Samsung washingMachine wash ...")
    }
}
```

6、HaierWashingMachine.swift

```swift
class HaierWashingMachine: WashingMachine {
    func wash() {
        print("Haier washingMachine wash ...")
    }
}
```


<!-- ************************************************ -->
## <a id="content4"></a>使用

**一、调用**

```swift
//我们代码内部使用产品族的各个实例处理事务
//当我们新增加一个产品族时，只需要创建一个产品族工厂，用工厂生产出产品族
func productFamilyEvent(productFamily: (tv:Televison,fri:Fridge,was:WashingMachine) ){
    productFamily.tv.show()
    productFamily.fri.cool()
    productFamily.was.wash()
}


//传入海尔工厂的产品族
let fac : Factory = HaierFactory()
let family = (tv: fac.createTelevison(),fri: fac.createFridge(),was: fac.createWashingMachine())
productFamilyEvent(productFamily:family)


//传入三星工厂的产品族
let fac2 = SamsungFactory()
let family2 = (fac2.createTelevison(),fac2.createFridge(),fac2.createWashingMachine())
productFamilyEvent(productFamily: family2)


//传入新增工厂的产品族
//...
```


**二、打印信息**

```swift
Haier televison showing ...
Haier Fridge cooling ...
Haier washingMachine wash ...
Samsung televison showing ...
Samsung fridge cooling ...
Samsung washingMachine wash ...
```





<!-- ************************************************ -->
## <a id="content5"></a>总结

优点：

抽象工厂在增加一个产品族工厂时很容易，扩展性也很好，比如我们再增加一个格力工厂，格力工厂可以生产格力电视、格力冰箱、格力洗衣机。当我们需要使用这些产品时，直接将该产品族(GeliTelevison,GeliFridge,GeliWashingMachine)应用于我们的代码即可，不需要原有代码进行改动。

缺点：

但当我们增加一个产品类别时，就显得很棘手，比如我们要增加一个空调，那么就需要改造所有的工厂，以及使用产品的代码，让它能够使用空调。






----------
>  行者常至，为者常成！


