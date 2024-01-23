---
layout: post
title: "WebViewJavaScriptBridge "
date: 2018-03-12
tag: Objective-C
--- 

- [参考：一篇文章了解JsBridge之IOS篇](https://juejin.cn/post/6844903567992553480?from=search-suggest)
- [参考：WebViewJavaScriptBridge 基本使用](https://www.jianshu.com/p/d12ec047ce52)
- [参考：iOS之WebViewJavascriptBridge浅析](https://juejin.cn/post/7168824876059328548)
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

#### **一、框架结构**   

|层级|	源文件|
|:----|:----|
|接口层|	WebViewJavascriptBridge && WKWebViewJavascriptBridge|
|实现层|	WebViewJavascriptBridgeBase|
|JS层|	WebViewJavascriptBridge_JS|


WebViewJavascriptBridge && WKWebViewJavascriptBridge 作为接口层主要负责提供方便的接口，隐藏实现细节         
WebViewJavascriptBridgeBase 实现细节都是通过该层去做的           
WebViewJavascriptBridge_JS 作为 JS 层存储了一段JS代码，在需要的时候注入到WebView中，最终实现 Native 与 JS 的交互。            
 

**1、接口层认识**   

因为WKWebView已经成为主流，这里只介绍WKWebViewJavascriptBridge，<span style="color:red;font-weight:bold;">但WebViewJavascriptBridge内相关的兼容处理还是很值得借鉴的。</span>

```objc
#if (__MAC_OS_X_VERSION_MAX_ALLOWED > __MAC_10_9 || __IPHONE_OS_VERSION_MAX_ALLOWED >= __IPHONE_7_1)
#define supportsWKWebView
#endif

#if defined supportsWKWebView

#import <Foundation/Foundation.h>
#import "WebViewJavascriptBridgeBase.h"
#import <WebKit/WebKit.h>

@interface WKWebViewJavascriptBridge : NSObject<WKNavigationDelegate, WebViewJavascriptBridgeBaseDelegate>

// 初始化
+ (instancetype)bridgeForWebView:(WKWebView*)webView;

// 开启日志
+ (void)enableLogging;

// 注册 handler (Native)
- (void)registerHandler:(NSString*)handlerName handler:(WVJBHandler)handler;

// 删除 handler (Native)
- (void)removeHandler:(NSString*)handlerName;

// 调用 handler (JS)
- (void)callHandler:(NSString*)handlerName;
- (void)callHandler:(NSString*)handlerName data:(id)data;
- (void)callHandler:(NSString*)handlerName data:(id)data responseCallback:(WVJBResponseCallback)responseCallback;

- (void)reset;

// 设置 webViewDelegate
- (void)setWebViewDelegate:(id)webViewDelegate;

// 禁用 JS AlertBox 的安全时长来加速消息传递，不推荐使用
- (void)disableJavscriptAlertBoxSafetyTimeout;

@end

#endif
```

**2、实现层认识**   

```objc
#import <Foundation/Foundation.h>

// 几个重要的宏定义
#define kOldProtocolScheme @"wvjbscheme"
#define kNewProtocolScheme @"https"
#define kQueueHasMessage   @"__wvjb_queue_message__"
#define kBridgeLoaded      @"__bridge_loaded__"

// 几个重要的类型：消息类型、回调类型、注册handler类型
typedef void (^WVJBResponseCallback)(id responseData);// 回调 block
typedef void (^WVJBHandler)(id data, WVJBResponseCallback responseCallback);// 注册的 Handler block
typedef NSDictionary WVJBMessage;// 消息类型 - 字典


// 执行js代码的协议
@protocol WebViewJavascriptBridgeBaseDelegate <NSObject>
- (NSString*) _evaluateJavascript:(NSString*)javascriptCommand;
@end


@interface WebViewJavascriptBridgeBase : NSObject

// 代理，指向接口层类，用以给对应接口绑定的 WebView 组件发送执行 JS 消息
@property (weak, nonatomic) id <WebViewJavascriptBridgeBaseDelegate> delegate;

// 启动消息队列，可以理解为存放 WVJBMessage
@property (strong, nonatomic) NSMutableArray* startupMessageQueue;

// 回调 blocks 字典，存放 WVJBResponseCallback 类型的 block
@property (strong, nonatomic) NSMutableDictionary* responseCallbacks;

// 已注册的 handlers 字典，存放 WVJBHandler 类型的 block
@property (strong, nonatomic) NSMutableDictionary* messageHandlers;

// 没卵用
@property (strong, nonatomic) WVJBHandler messageHandler;

// 开启日志
+ (void)enableLogging;
// 设置日志最大长度
+ (void)setLogMaxLength:(int)length;

// 对应 WKJSBridge 的 reset 接口
- (void)reset;

// 发送消息，入参依次是参数，回调 block，对应 JS 端注册的 HandlerName
- (void)sendData:(id)data responseCallback:(WVJBResponseCallback)responseCallback handlerName:(NSString*)handlerName;

// 刷新消息队列，核心代码
- (void)flushMessageQueue:(NSString *)messageQueueString;

// 注入 JS
- (void)injectJavascriptFile;

// 判定是否为 WebViewJavascriptBridgeURL
- (BOOL)isWebViewJavascriptBridgeURL:(NSURL*)url;

// 判定是否为队列消息 URL
- (BOOL)isQueueMessageURL:(NSURL*)urll;

// 判定是否为 bridge 载入 URL
- (BOOL)isBridgeLoadedURL:(NSURL*)urll;

// 打印收到未知消息信息
- (void)logUnkownMessage:(NSURL*)url;

// JS bridge 检测命令
- (NSString *)webViewJavascriptCheckCommand;

// JS bridge 获取查询命令
- (NSString *)webViewJavascriptFetchQueyCommand;

// 禁用 JS AlertBox 安全时长以获取发送消息速度提升，不建议使用，理由见上文
- (void)disableJavscriptAlertBoxSafetyTimeout;
@end
```

**3、js层**   
js层是一段通过`;(function(){})()`自执行的js代码，转为字符串后，在合适的时机通过wkwebview.evaluateJavascript的方式注入到js环境中。             
关于 WebViewJavascriptBridge_JS 内部的 JS 代码我们放到后面的章节解读，现在可以简单理解为 WebViewJavascriptBridge 在 JS 端的具体实现代码。       



#### **二、js调用原生的核心逻辑**   





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






----------
>  行者常至，为者常成！



