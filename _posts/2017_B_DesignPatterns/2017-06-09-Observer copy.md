---
layout: post
title: "9、Observer"
date: 2017-06-09
description: ""
tag: 设计模式
---


- [参考：观察者模式](https://www.runoob.com/design-pattern/observer-pattern.html)





## 目录

* [观察者模式](#content1)
* [代码实现](#content2)




<!-- ************************************************ -->
## <a id="content1"></a>观察者模式

当对象间存在一对多关系时，则使用观察者模式（Observer Pattern）。比如，当一个对象被修改时，则会自动通知依赖它的对象。观察者模式属于行为型模式。

主要解决了:一个对象状态改变给其他对象通知的问题，而且要考虑到易用和低耦合，保证高度的协作。

关键代码：在抽象类里有一个 ArrayList 存放观察者们。

可以使用观察者模式创建一种链式触发机制,需要在系统中创建一个触发链，A对象的行为将影响B对象，B对象的行为将影响C对象……，。

<img src="/images/DesignPatterns/observer.png" alt="img">


<!-- ************************************************ -->
## <a id="content2"></a>代码实现

**一 代码实现**

1 OBEvent.swift

```swift
class OBEvent {
    var time:CLong = 0
    var location:String = ""
    var source:Child? = nil
}


extension OBEvent : CustomStringConvertible {
    var description: String {
        "time:\(time),location:\(location),source:\(source)"
    }
}
```

2 Child.swift

```swift
class Child: NSObject {
    private var wakeUpListeners:[WakeUpListener] = []
    private var status : Int = 0 //0:睡觉 1:醒来
    
    func addWakeUpListener(listener:WakeUpListener) {
        wakeUpListeners.append(listener)
    }
    
    func removeWakeUpListener(listenId:String) {
        wakeUpListeners = wakeUpListeners.filter { (listener) -> Bool in
            listener.listenId != listenId
        }
    }
    
    func setStatus(status:Int) {
        if self.status == status { return }
        self.status = status
        if self.status == 0 { return }
        for listener in wakeUpListeners {
            let event = OBEvent()
            event.time = 2000
            event.location = "家中卧室"
            event.source = self
            listener.actionForWakeUp(event: event)
        }
    }
    
    func getStatus() -> Int {
        status
    }
}
```


3 WakeUpListener.swift

```swift
protocol WakeUpListener {
    var listenId : String {set get}
    func actionForWakeUp(event:OBEvent) 
}
```

4 Dad.swift

```swift
class Dad: WakeUpListener {
    var listenId: String
    
    func actionForWakeUp(event: OBEvent) {
//        print("Dad:",event)
        
        print("Dad play ... ")
    }
    
    init(listenId:String) {
        self.listenId = listenId
    }
}
```

5 Mum.swift

```swift
class Mum: WakeUpListener {
    var listenId: String
    
    func actionForWakeUp(event: OBEvent) {
//        print("Mum:",event)
        
        print("Mum feed ...")
    }
    
    init(listenId:String) {
        self.listenId = listenId
    }
}

```


**二 调用**

```swift
    func test1() {
        
        let child = Child()
        let mum = Mum(listenId: "mumId")
        child.addWakeUpListener(listener: mum)
        child.setStatus(status: 1)
        
        print("添加一个监听者")
        let dad = Dad(listenId: "dadId")
        child.addWakeUpListener(listener: dad)
        child.setStatus(status: 0)
        child.setStatus(status: 1)
        
        print("移除一个监听者")
        child.removeWakeUpListener(listenId: mum.listenId)
        child.setStatus(status: 0)
        child.setStatus(status: 1)
        
        /**
         Mum feed ...
         添加一个监听者
         Mum feed ...
         Dad play ...
         移除一个监听者
         Dad play ...
         */
    }
```







----------
>  行者常至，为者常成！


