---
layout: post
title: "2、SimpleFactory"
date: 2020-12-04
description: ""
tag: 设计模式
---






## 目录

* [什么是工厂模式](#content1)
* [简单工厂模式实现](#content2)
* [总结](#content3)






<!-- ************************************************ -->
## <a id="content1"></a>什么是工厂模式

**一、工厂模式**

工厂模式也是最常用的实例化对象模式了，是用工厂方法代替 new 操作的一种模式。但为什么有了初始化方法，还要有工厂模式？

1、是为了控制生产过程，并将生产和使用隔离开。实际开发过程中创建一个产品对象的逻辑可能很复杂，这时我们把这个过程交给工厂。使用时并不关心产品对象是怎么创建的。

2、便于扩展，使用工厂模式当我们添加一个新的产品对象时，并不会影响之前的逻辑。开闭原则，对扩展开放，对修改关闭

所以工厂模式也是用来创建实例对象的，所以以后 new 时就要多个心眼，是否可以考虑使用工厂模式，虽然这样做，可能多做一些工作，但会给你系统带来更大的可扩展性和尽量少的修改量。

**二、工厂模式分类**


简单工厂模式

工厂模式

抽象工厂模式


<!-- ************************************************ -->
## <a id="content2"></a>简单工厂模式实现



**一、创建一个接口**

```swift
protocol CardWatchCenter {
    
    /// 设置获取闹钟
    var clocks : [(hour:Int,minute:Int,second:Int)]{ get set }
    
    
    /// 获取步数
    var step : Int { get }
    
    
    /// 获取心率
    var heartRate : Int{ get }
    
    
    /// 连接设备
    /// - Parameter uuid: 设备的uuid
    @discardableResult
    func connect(uuid:String)->Bool
    
    
    /// 断开当前设备
    @discardableResult
    func disConnect()->Bool
}
```

**二、创建实现接口的实体类**

B1Center.swift

```swift
class B1Center: CardWatchCenter {
    var clocks: [(hour: Int, minute: Int, second: Int)]{
        
        get{
            let clock1 = (hour:9,minute:0,second:0)
            let clock2 = (hour:22,minute:0,second:0)
            return [clock1,clock2]
        }
        
        
        set{
            guard newValue.count > 1 else {return}
            let clock = newValue.first!
            let _  = clock.hour
            let _  = clock.minute
            let _  = clock.second
        }
    }
    
    var step: Int {
        100
    }
    
    var heartRate: Int {
        80
    }
    
    func connect(uuid: String) -> Bool {
        print("B1Center : connet device uuid \(uuid)")
        return true
    }
    
    func disConnect() -> Bool {
        print("B1Center : disconnect current device")
        return true
    }
}
```

B2Center.swift
```swift
class B2Center: CardWatchCenter {
    var clocks: [(hour: Int, minute: Int, second: Int)]{
        
        get{
            let clock1 = (hour:9,minute:1,second:1)
            let clock2 = (hour:22,minute:1,second:1)
            return [clock1,clock2]
        }
        
        
        set{
            guard newValue.count > 1 else {return}
            let clock = newValue.first!
            let _  = clock.hour
            let _  = clock.minute
            let _  = clock.second
        }
    }
    
    var step: Int {
        101
    }
    
    var heartRate: Int {
        81
    }
    
    func connect(uuid: String) -> Bool {
        print("B2Center : connet device uuid \(uuid)")
        return true
    }
    
    func disConnect() -> Bool {
        print("B2Center : disconnect current device")
        return true
    }
}
```

**三、创建一个工厂**

创建一个工厂，生成基于给定信息的实体类的对象。

```swift
class CardWatchCenterFactory: NSObject {

    enum CenterType {
        case CenterTypeB1
        case CenterTypeB2
    }
    
    
    func createCenter(type:CenterType) -> CardWatchCenter {
        
        switch type {
        
        case .CenterTypeB1:
            
            //创建过程可能很复杂...
            return B1Center()
            
        case .CenterTypeB2:
            
            //创建过程可能很复杂...
            return B2Center()
        }
    }
}
```

**四、使用该工厂**

使用该工厂，通过传递类型信息来获取实体类的对象。

```swift
//我们代码内部使用CardWatchCenter实例处理事务
//当我们新增加一个产品时，只需要创建一个产品类，并在工厂类内增加类型和创建即可，下面代码不需要改动
func cardWatchCenterEvent(_ center : CardWatchCenter){
    
    let clocks = center.clocks
    print(clocks)

    let heartRate = center.heartRate
    print(heartRate)
    
    let step = center.step
    print(step)
    
    center.connect(uuid: "xxxx-yyyyy-ttttt")
    center.disConnect()
}


cardWatchCenterEvent(CardWatchCenterFactory().createCenter(type: .CenterTypeB1))
print("----------")
cardWatchCenterEvent(CardWatchCenterFactory().createCenter(type: .CenterTypeB2))
```

**五、打印信息**

```swift
[(hour: 9, minute: 0, second: 0), (hour: 22, minute: 0, second: 0)]
80
100
B1Center : connet device uuid xxxx-yyyyy-ttttt
B1Center : disconnect current device
----------
[(hour: 9, minute: 1, second: 1), (hour: 22, minute: 1, second: 1)]
81
101
B2Center : connet device uuid xxxx-yyyyy-ttttt
B2Center : disconnect current device
```


<!-- ************************************************ -->
## <a id="content3"></a>总结

下面有几条关于简单工厂需要记住的东西。

1、它通过分离 构造 和 使用逻辑 实现松散耦合。   

2、它只是一种对于经常变化的事情的包装器。

3、Swift 里的简单工厂可以采用 enum 和 switch-case 来实现。

4、如果你想返回不同的对象，可以使用协议。(POP )

5、保持简单

这个设计模式将创建过程和实际使用分离开，将创建的责任移给特定的角色。如果创建过程有变化，只需要改动工厂即可，其他代码可以完全不变。强大又简单！



----------
>  行者常至，为者常成！


