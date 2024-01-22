---
layout: post
title: "WebViewJavaScriptBridge "
date: 2018-03-12
tag: Objective-C
--- 

- [参考：一篇文章了解JsBridge之IOS篇](https://juejin.cn/post/6844903567992553480?from=search-suggest)
- [参考：WebViewJavaScriptBridge 基本使用](https://www.jianshu.com/p/d12ec047ce52)
- [重要：深入剖析 WebViewJavascriptBridge](https://lision.me/webview_javascript_bridge/)     




## 目录
* [介绍](#content1)
* [基本使用](#content2)
* [三方库 WebViewJavascriptBridge 原理分析](#content3)





<!-- ************************************************ -->
## <a id="content1">介绍</a>

WebViewJavaScriptBridge 用于 WKWebView & UIWebView 中 OC 和 JS 交互的框架。    

把 OC 的方法注册到桥梁中，让 JS 去调用。

把 JS 的方法注册在桥梁中，让 OC 去调用。



<!-- ************************************************ -->
## <a id="content2">基本使用</a>

#### **一、使用pod管理**  

```ruby
pod ‘WebViewJavascriptBridge’
```

#### **二、OC注册方法(JS调用)**

**1、OC如何代码书写**     
```objc
// 引用头文件
#import <WebViewJavascriptBridge/WebViewJavascriptBridge.h>

// 初始化
self.jsBridge = [WebViewJavascriptBridge bridgeForWebView:self.wkWebView];

// 注册方法：让js来调用。
[self.jsBridge registerHandler:@"colorClick" handler:^(id param, WVJBResponseCallback responseCallback) {
    
    // xy:数据类型的对应可以查看jsvalue
    NSLog(@"param From JS: %@",param);
    
    UIColor *randomColor = [UIColor colorWithRed:arc4random_uniform(256) / 255.0
                                            green:arc4random_uniform(256) / 255.0
                                                blue:arc4random_uniform(256) / 255.0
                                            alpha:1.0];
    
    self.view.backgroundColor = randomColor;
    
    // xy:数据类型的对应可以查看jsvalue
    responseCallback(@"oc return success");
}];
```

**2、JS如何调用**     

```css
<div class="container" onclick="changeColor()">js调用OC:修改view背景色</div>
```

```js
// WebViewJavascriptBridge 是需要在H5页面加载时创建的对象，后边会解释
function changeColor() {
    WebViewJavascriptBridge.callHandler(
        // 调用的方法名
        'colorClick',
        // 调用方法传递的参数
        {color:'green'},
        // 通过回调函数获取方法的返回值
        function(data) {
            //获取返回值
            document.getElementById("returnValue").innerHTML = data;
        }
    )
}
```

#### **三、JS注册方法(OC调用)**

**1、JS如何代码书写** 

```js
function setupWebViewJavascriptBridge(callback) {
    
    if (window.WebViewJavascriptBridge) { return callback(WebViewJavascriptBridge); }
    
    if (window.WVJBCallbacks) { return window.WVJBCallbacks.push(callback); }
    
    // 创建一个 WVJBCallbacks 全局属性数组，并将 callback 插入到数组中。
    window.WVJBCallbacks = [callback];
    
    // 创建一个 iframe 元素
    var WVJBIframe = document.createElement('iframe');
    // 不显示
    WVJBIframe.style.display = 'none';
    
    // 设置 iframe 的 src 属性
    WVJBIframe.src = 'wvjbscheme://__BRIDGE_LOADED__';
    
    // 把 iframe 添加到当前文导航上。
    document.documentElement.appendChild(WVJBIframe);
    
    setTimeout(function() { document.documentElement.removeChild(WVJBIframe) }, 0)
}


// 这里主要是注册 OC 将要调用的 JS 方法。
setupWebViewJavascriptBridge(function(bridge){
    // 声明 OC 需要调用的 JS 方法。
    bridge.registerHandler('changeBGColor',function(data,responseCallback){
        // data 是 OC 传递过来的参数
        
        // 修改颜色
        var randomColor = generateRandomColor();
        document.body.style.backgroundColor = randomColor
        
        // responseCallback 是 JS 调用完毕之后传递给 OC 的返回值
        responseCallback('js return ' + randomColor);
    });
});


// 调用oc方法colorClick
function changeColor() {
    WebViewJavascriptBridge.callHandler(
        'colorClick',
        {color:'green'},
        function(data) {
            //获取返回值
            document.getElementById("returnValue").innerHTML = data;
        }
    )
}


function generateRandomColor() {
    // 生成随机的RGB值
    var red = Math.floor(Math.random() * 256);
    var green = Math.floor(Math.random() * 256);
    var blue = Math.floor(Math.random() * 256);

    // 将RGB值转换为十六进制格式
    var hexRed = red.toString(16).padStart(2, '0');
    var hexGreen = green.toString(16).padStart(2, '0');
    var hexBlue = blue.toString(16).padStart(2, '0');

    // 拼接十六进制颜色码
    var hexColor = '#' + hexRed + hexGreen + hexBlue;

    return hexColor;
}
```

**2、OC如何调用**  
```objc
// oc调用js方法
- (IBAction)changeHTMLBgColor:(id)sender {
    // callHandler:要调用的jd的方法changeBGColor
    // data:方法调用要传递的参数，参数类型参考jsvalue
    // responseCallback:通过回调函数获取方法的返回值
    [self.jsBridge callHandler:@"changeBGColor" data:@{@"color":@"orange"} responseCallback:^(id responseData) {
        self.valueLabel.text = responseData;
    }];
}
```

<!-- ************************************************ -->
## <a id="content3">三方库 WebViewJavascriptBridge 原理分析</a>

- [参考：iOS之WebViewJavascriptBridge浅析](https://juejin.cn/post/7168824876059328548)


在js的`changeColor`方法中，其实是调用了`WebViewJavascriptBridge.callHandler`方法，那么`WebViewJavascriptBridge`对象是在什么地方创建的呢？    
在js代码中并没有找到相关代码，在OC代码中我们找到了相关的代码     

```objc
- (void)injectJavascriptFile {
    NSString *js = WebViewJavascriptBridge_js();
    [self _evaluateJavascript:js];
    if (self.startupMessageQueue) {
        NSArray* queue = self.startupMessageQueue;
        self.startupMessageQueue = nil;
        for (id queuedMessage in queue) {
            [self _dispatchMessage:queuedMessage];
        }
    }
}
```

**几个重要的宏定义**  
```objc
#define kOldProtocolScheme @"wvjbscheme"
#define kNewProtocolScheme @"https"
#define kQueueHasMessage   @"__wvjb_queue_message__"
#define kBridgeLoaded      @"__bridge_loaded__"
```




----------
>  行者常至，为者常成！



