---
layout: post
title: "layoutSubviews"
date: 2017-06-02
tag: Objective-C
---





## 目录
* [官方文档](#content1)
* [调用时机](#content2)
* [刷新标记](#content3)



<!-- ************************************************ -->
## <a id="content1">官方文档</a>

- (void)layoutSubviews;


The default implementation of this method does nothing on iOS 5.1 and earlier. Otherwise, the default implementation uses any constraints you have set to determine the size and position of any subviews.
Subclasses can override this method as needed to perform more precise layout of their subviews. You should override this method only if the autoresizing and constraint-based behaviors of the subviews do not offer the behavior you want. You can use your implementation to set the frame rectangles of your subviews directly.
You should not call this method directly. If you want to force a layout update, call the setNeedsLayout() method instead to do so prior to the next drawing update. If you want to update the layout of your views immediately, call the layoutIfNeeded() method.


在ios5.1和更早的版本中，这个方法的默认实现什么都不做。

否则，默认实现使用您设置的任何约束来确定任何子视图的大小和位置。

子类可以根据需要重写这个方法，以执行更精确的子视图布局。

只有当子视图的自动调整大小和基于约束的行为不能提供您想要的行为时，才应该重写此方法。你可以使用你的实现直接设置子视图的框架矩形。

你不应该直接调用这个方法。如果你想强制一个布局更新，在下一次绘图更新之前调用setNeedsLayout()方法。如果您想立即更新视图的布局，请调用layoutIfNeeded()方法。


<!-- ************************************************ -->
## <a id="content2">调用时机</a>

  
自身被添加到父view上的时候会调用,自身被移除时不会调用
    
自身的x和y改变的时候不会调用,大小改变的时候会调用

添加或移除子view的时候会调用

更改子view的x和y时不会调用,更改子view的大小的时候会调用

思考：单纯的移动不会引起布局的递归更改，但改变大小有可能引起布局的递归更改，是否是这个原因？

<!-- ************************************************ -->
## <a id="content3">刷新标记</a>

```
[self setNeedsLayout];
NSLog(@"xxx");

打印日志
xxx
-[TestView layoutSubviews]
```

从打印日志可以看出，是在下一个刷新周期调用的layoutSubviews


```  
[self setNeedsLayout];
[self layoutIfNeeded];
NSLog(@"xxx");

打印日志
-[TestView layoutSubviews]
xxx
```

从打印日志可以看出，是立即调用的layoutSubviews

setNeedsLayout 是设置刷新标记，layoutIfNeeded是有刷新标记时就立即调用layoutSubviews。

如果只是[self layoutIfNeeded];不会调用layoutSubviews，因为没有标记




----------
>  行者常至，为者常成！


