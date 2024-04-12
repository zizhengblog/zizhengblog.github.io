---
layout: post
title: "UniversalLinks"
date: 2018-05-02
tag: Objective-C
---


- [参考文章：iOS Universal Link使用总结](https://juejin.cn/post/7011427577164202015?from=search-suggest)
- [参考文章：iOS配置Universal Links](https://www.jianshu.com/p/1910ea1fe8f6)



## 目录
- [介绍](#content1) 
- [如何配置](#content2) 
- [原理分析](#content3) 
- [微信SDK集成](#content4) 



<!-- ************************************************ -->
## <a id="content1">介绍</a>


Universal Link是Apple在WWDC上提出的iOS9的新特性之一；    
能够方便的通过打开一个HTTPS链接来直接启动您的客户端应用，当然前提条件是您必须在手机上安装了此App；  

**应用场景：**       
1、从浏览器启动你的app      
2、从其他App的WebView中拉起你的App      
3、从其它app中拉起你的App    



<!-- ************************************************ -->
## <a id="content2">如何配置</a>

第一步：开发者网站配置    
找到”Associated Domains“选项，打开勾选；   

第二步：配置服务器         
在host域名对应的服务器.well-known目录下面新建一个json文件，命名为apple-app-site-association，不需要.json后缀    
测试办法：例如你的{host}是mcrm.founder.com，那浏览器打开https://mcrm.founder.com/apple-app-site-association可以下载该文件即可    
   

```json
{
  "applinks": {
      "apps": [],
      "details": [
          {
              // teamid+bundleId
              "appID":"V6AMULB632.com.founder.xxx.mobile",
              // 标识了具体的app，此时该app的UniversalLink是：https://mcrm.founder.com/redirectApp/
              "paths": [ "/redirectApp/*"]
          },
          {
            "appID":"85YY7C36JS.com.founder.yyy.mobile",
            "paths": [ "/redirectApp/*"]
          }
      ]
  }
}
```


第三步：配置Xcode Project    

打开你的iOS工程，在project → Signing & Capabilities → + Capability，添加“Associated Domains”   
添加配置，**applinks:mcrm.founder.com**   


第四步：测试UL链接    

根据服务器配置目录打开 https://mcrm.founder.com/apple-app-site-association 或者 https://mcrm.founder.com/apple-app-site-association，   
检查是否包含应用Bundle ID，   可以参考微信的apple-app-site-association：   
<a href="https://help.wechat.com/apple-app-site-association">https://help.wechat.com/apple-app-site-association</a>

也可以参考这个
<a href="https://xdf.fzzqft.com/redirectApp/">https://xdf.fzzqft.com/redirectApp/</a>



<!-- ************************************************ -->
## <a id="content3">原理分析</a>

准确来说，iOS系统是在安装app应用的过程中向服务器发起请求，如果获得数据则会导入系统   
系统会优先请求：https://{host}/.well-known/apple-app-site-association    
如果第1步没请求成功，才会请求：https://{host}/apple-app-site-association     
lxy：在xcode配置的applinks:mcrm.founder.com可以帮助找到文件，并进行下载，文件拿到后将数据写入系统，别的app就可以调起当前的app。        

lxy：微信SDK，需要在微信开放平台填写app的universal link,在sdk初始化的时候universal link也需要作为参数传入，只有拿到universal link微信才能调起我们的app    



<!-- ************************************************ -->
## <a id="content4">微信SDK集成</a>

微信SDK的分享，调起小程序，现在使用的是Universal links的方式，所以在使用微信SDK提供的功能之前，需要完成上面提到的各项配置    
关于微信SDK的集成和初始化，微信官方有详细的文档说明，并根据SDK提供的接口可以进行自检      

在集成时遇到始终无法自检通过且无法正常调起微信小程序，原因是缺少了下面的配置

<img src="/images/objectC/objc20.jpeg" alt="img">

 





----------
>  行者常至，为者常成！


