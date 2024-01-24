---
layout: post
title: "WebViewJavaScriptBridge原理分析"
date: 2018-03-13
tag: Objective-C
--- 

- [参考：iOS之WebViewJavascriptBridge浅析](https://juejin.cn/post/7168824876059328548)
- [重要：深入剖析 WebViewJavascriptBridge](https://lision.me/webview_javascript_bridge/)     




## 目录
* [框架结构](#content1)
* [js调用原生的核心逻辑](#content2)


<!-- ************************************************ -->
## <a id="content1">框架结构</a>


|层级|	源文件|
|:----|:----|
|接口层|	WebViewJavascriptBridge && WKWebViewJavascriptBridge|
|实现层|	WebViewJavascriptBridgeBase|
|JS层|	WebViewJavascriptBridge_JS|


WebViewJavascriptBridge && WKWebViewJavascriptBridge 作为接口层主要负责提供方便的接口，隐藏实现细节         
WebViewJavascriptBridgeBase 实现细节都是通过该层去做的           
WebViewJavascriptBridge_JS 作为 JS 层存储了一段JS代码，在需要的时候注入到WebView中，最终实现 Native 与 JS 的交互。            
 

#### **一、接口层认识**   

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

#### **二、实现层认识**   

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

#### **三、js层**   
js层是一段`;(function(){})()`包裹的js代码，转为字符串后，在合适的时机通过wkwebview.evaluateJavascript的方式在js环境中执行。               
关于 WebViewJavascriptBridge_JS 内部的 JS 代码我们放到后面的章节解读，现在可以简单理解为 WebViewJavascriptBridge 在 JS 端的具体实现代码。         



<!-- ************************************************ -->
## <a id="content2">js调用原生的核心逻辑</a>

#### **一、js 端的初始化操作**    

在使用这个框架时web端需要拷贝一段js代码，这段代码如下：

```js
// 定义一个js方法
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

// 调用这个 JS 方法。
setupWebViewJavascriptBridge(function(bridge){
    // 在这里 声明 OC 需要调用的 JS 方法。
   
});
```

上面代码的本质是<span style="color:red;font-weight:bold;">使用iframe加载一个固定格式的url：`wvjbscheme://__BRIDGE_LOADED__` </span>        

使用自定义的scheme `wvjbscheme`来与普通的url进行区分,在最新版本中也是支持`https`协议头的          
使用自定义的host`__BRIDGE_LOADED__`来区分是load 还是 message     
如果是发送message的话host部分是：`__wvjb_queue_message__`       

总结下来就是：   
scheme可以是：`wvjbscheme` 和 `https`     
host可以是：`__BRIDGE_LOADED__` 和 `__wvjb_queue_message__`     
这也正好对应上边提到的原生端的几个关键的宏定义     
```objc
// 几个重要的宏定义
#define kOldProtocolScheme @"wvjbscheme"
#define kNewProtocolScheme @"https"
#define kQueueHasMessage   @"__wvjb_queue_message__"
#define kBridgeLoaded      @"__bridge_loaded__"
```          

原生端通过<span style="color:red;font-weight:bold;">拦截iframe加载的url，</span>并通过判断不同的scheme 和 host来进行不同的操作          

```objc
- (void)webView:(WKWebView *)webView decidePolicyForNavigationAction:(WKNavigationAction *)navigationAction decisionHandler:(void (^)(WKNavigationActionPolicy))decisionHandler {
    if (webView != _webView) { return; }
    NSURL *url = navigationAction.request.URL;
    __strong typeof(_webViewDelegate) strongDelegate = _webViewDelegate;

    // 一、判定是否是 WebViewJavascriptBridge 格式的URL
    // (scheme是：wvjbscheme或者https 并且 host是__BRIDGE_LOADED__或者__BRIDGE_LOADED__)      
    if ([_base isWebViewJavascriptBridgeURL:url]) {

        // 判定 BridgeLoadedURL: wvjbscheme://__BRIDGE_LOADED__ 或者 https://__BRIDGE_LOADED__
        if ([_base isBridgeLoadedURL:url]) {
            [_base injectJavascriptFile];

        // 判定 QueueMessageURL: wvjbscheme://__wvjb_queue_message__ 或者 https://__wvjb_queue_message__  
        } else if ([_base isQueueMessageURL:url]) {
            [self WKFlushMessageQueue];

        // 记录未知 bridge msg 日志
        } else {
            [_base logUnkownMessage:url];
        }

        //  WebViewJavascriptBridge 格式的URL 是用来传递数据的不是真正的要加载url，所以取消跳转     
        decisionHandler(WKNavigationActionPolicyCancel);
        return;
    }
    

    // 二、下面是对普通url的处理    

    // 如果设置了代理，交给代理去处理,这里相当于hook了代理方法  
    if (strongDelegate && [strongDelegate respondsToSelector:@selector(webView:decidePolicyForNavigationAction:decisionHandler:)]) {
        [_webViewDelegate webView:webView decidePolicyForNavigationAction:navigationAction decisionHandler:decisionHandler];

    // 直接进行加载跳转处理
    } else {
        decisionHandler(WKNavigationActionPolicyAllow);
    }
}
```

如果是BridgeLoadedURL会执行下面的代码     
```objc
- (void)injectJavascriptFile {
    // 获取到 WebViewJavascriptBridge_JS 的代码
    // 这段js代码内会创建WebViewJavascriptBridge对象     
    NSString *js = WebViewJavascriptBridge_js();

    // 将获取到的 js 通过代理方法注入到当前绑定的 WebView 组件
    [self _evaluateJavascript:js];

    // 如果当前已有消息队列则遍历并分发消息，之后清空消息队列
    if (self.startupMessageQueue) {
        NSArray* queue = self.startupMessageQueue;
        self.startupMessageQueue = nil;
        for (id queuedMessage in queue) {
            [self _dispatchMessage:queuedMessage];
        }
    }
}
```

该步骤的主要作用就是向js环境中注入一段js代码来初始化js环境，到此为止，js端的相关环境已经准备完成了。          

#### **二、调用原生端**   

执行下面代码来调用原生端的colorClick方法     
```js
// 调用oc方法colorClick
function changeColor() {
    WebViewJavascriptBridge.callHandler(
        'colorClick',       // 方法名
        {color:'green'},    // 传递的参数
        function(data) {    // 通过回调获取返回值
            document.getElementById("returnValue").innerHTML = data;
        }
    )
}
```

我们看看callHandler的实现   
```js
// 调用 iOS handler，参数校验之后调用 _doSend 函数
function callHandler(handlerName, data, responseCallback) {
	if (arguments.length == 2 && typeof data == 'function') {
		responseCallback = data;
		data = null;
	}
	_doSend({ handlerName:handlerName, data:data }, responseCallback);
}

// 如有回调，则设置 message['callbackId'] 与 responseCallbacks[callbackId]
// 将 msg 加入 sendMessageQueue 数组，设置 messagingIframe.src
function _doSend(message, responseCallback) {
    if (responseCallback) {
        var callbackId = 'cb_'+(uniqueId++)+'_'+new Date().getTime();
        responseCallbacks[callbackId] = responseCallback;
        message['callbackId'] = callbackId;
    }
    sendMessageQueue.push(message);
    messagingIframe.src = CUSTOM_PROTOCOL_SCHEME + '://' + QUEUE_HAS_MESSAGE;
}
	
// scheme 使用 https 之后通过 host 做匹配
var CUSTOM_PROTOCOL_SCHEME = 'https';
var QUEUE_HAS_MESSAGE = '__wvjb_queue_message__';
```
<span style="color:red;">这里将message放入了sendMessageQueue，将responseCallback放入了responseCallbacks</span>      

原生端拦截到`https://__wvjb_queue_message__`会执行下面的方法  

```objc
- (void)WKFlushMessageQueue {
    [_webView evaluateJavaScript:[_base webViewJavascriptFetchQueyCommand] completionHandler:^(NSString* result, NSError* error) {
        if (error != nil) {
            NSLog(@"WebViewJavascriptBridge: WARNING: Error when trying to fetch data from WKWebView: %@", error);
        }
        [_base flushMessageQueue:result];
    }];
}
```

```objc
- (NSString *)webViewJavascriptFetchQueyCommand {
    return @"WebViewJavascriptBridge._fetchQueue();";
}
```

```js
function _fetchQueue() {
    var messageQueueString = JSON.stringify(sendMessageQueue);
    sendMessageQueue = [];
    return messageQueueString;
}
```
这是原生端处理js发送过来的消息的<span style="color:red;font-weight:bold;">核心代码</span>
```objc
- (void)flushMessageQueue:(NSString *)messageQueueString{
    if (messageQueueString == nil || messageQueueString.length == 0) {
        NSLog(@"WebViewJavascriptBridge: WARNING: ObjC got nil while fetching the message queue JSON from webview. This can happen if the WebViewJavascriptBridge JS is not currently present in the webview, e.g if the webview just loaded a new page.");
        return;
    }

    // 将json数组字符串反序列化为数组
    id messages = [self _deserializeMessageJSON:messageQueueString];

    for (WVJBMessage* message in messages) {
        if (![message isKindOfClass:[WVJBMessage class]]) {
            NSLog(@"WebViewJavascriptBridge: WARNING: Invalid %@ received: %@", [message class], message);
            continue;
        }
        [self _log:@"RCVD" json:message];
        
        NSString* responseId = message[@"responseId"];
        if (responseId) {
            WVJBResponseCallback responseCallback = _responseCallbacks[responseId];
            responseCallback(message[@"responseData"]);
            [self.responseCallbacks removeObjectForKey:responseId];
        } else {

            // 这里是处理js发送过来的消息

            WVJBResponseCallback responseCallback = NULL;
            NSString* callbackId = message[@"callbackId"];
            if (callbackId) {
                // 有返回值
                responseCallback = ^(id responseData) {
                    if (responseData == nil) {
                        responseData = [NSNull null];
                    }
                    
                    // msg 是返回给js端的方法调用返回值
                    WVJBMessage* msg = @{ @"responseId":callbackId, @"responseData":responseData };
                    [self _queueMessage:msg];
                };
            } else {
                // 无返回值
                responseCallback = ^(id ignoreResponseData) {
                    // Do nothing
                };
            }
            
            // 取出原生端注册的handler,其实就是block
            WVJBHandler handler = self.messageHandlers[message[@"handlerName"]];
            
            if (!handler) {
                NSLog(@"WVJBNoHandlerException, No handler for message from JS: %@", message);
                continue;
            }
            
            // 执行block
            handler(message[@"data"], responseCallback);
        }
    }
}
```
#### **三、如何获取调用原生端方法的返回值**      

我们在看看方法调用的返回值又是如何传递回给js端的？   

下面是_queueMessage的方法实现     
```objc
- (void)_queueMessage:(WVJBMessage*)message {
    if (self.startupMessageQueue) {
        [self.startupMessageQueue addObject:message];
    } else {
        [self _dispatchMessage:message];
    }
}


- (void)_dispatchMessage:(WVJBMessage*)message {
    NSString *messageJSON = [self _serializeMessage:message pretty:NO];
    [self _log:@"SEND" json:messageJSON];
    messageJSON = [messageJSON stringByReplacingOccurrencesOfString:@"\\" withString:@"\\\\"];
    messageJSON = [messageJSON stringByReplacingOccurrencesOfString:@"\"" withString:@"\\\""];
    messageJSON = [messageJSON stringByReplacingOccurrencesOfString:@"\'" withString:@"\\\'"];
    messageJSON = [messageJSON stringByReplacingOccurrencesOfString:@"\n" withString:@"\\n"];
    messageJSON = [messageJSON stringByReplacingOccurrencesOfString:@"\r" withString:@"\\r"];
    messageJSON = [messageJSON stringByReplacingOccurrencesOfString:@"\f" withString:@"\\f"];
    messageJSON = [messageJSON stringByReplacingOccurrencesOfString:@"\u2028" withString:@"\\u2028"];
    messageJSON = [messageJSON stringByReplacingOccurrencesOfString:@"\u2029" withString:@"\\u2029"];
    
    // messageJSON 是将返回值转化为了json字符串
    NSString* javascriptCommand = [NSString stringWithFormat:@"WebViewJavascriptBridge._handleMessageFromObjC('%@');", messageJSON];


    // 通过调用js端的_handleMessageFromObjC()方法，把字符串传递给js端
    if ([[NSThread currentThread] isMainThread]) {
        [self _evaluateJavascript:javascriptCommand];

    } else {
        dispatch_sync(dispatch_get_main_queue(), ^{
            [self _evaluateJavascript:javascriptCommand];
        });
    }
}
```


这里是js端的<span style="color:red;font-weight:bold;">核心代码</span>
```js
function _handleMessageFromObjC(messageJSON) {
    _dispatchMessageFromObjC(messageJSON);
}

function _dispatchMessageFromObjC(messageJSON) {
    if (dispatchMessagesWithTimeoutSafety) {
        setTimeout(_doDispatchMessageFromObjC);
    } else {
            _doDispatchMessageFromObjC();
    }
    
    function _doDispatchMessageFromObjC() {
        var message = JSON.parse(messageJSON);
        var messageHandler;
        var responseCallback;

        if (message.responseId) {

            // 重要：这里是处理原生端发送过来返回值

            // responseId也就是js端调用时的callbackId,通过这个id可以取出对应的匿名函数
            responseCallback = responseCallbacks[message.responseId];
            if (!responseCallback) {
                return;
            }

            // 调用匿名函数，传递返回值
            responseCallback(message.responseData);
            delete responseCallbacks[message.responseId];

        } else {
            if (message.callbackId) {
                var callbackResponseId = message.callbackId;
                responseCallback = function(responseData) {
                    _doSend({ handlerName:message.handlerName, responseId:callbackResponseId, responseData:responseData });
                };
            }
            
            var handler = messageHandlers[message.handlerName];
            if (!handler) {
                console.log("WebViewJavascriptBridge: WARNING: no handler for message from ObjC:", message);
            } else {
                handler(message.data, responseCallback);
            }
        }
    }
}
```













----------
>  行者常至，为者常成！



