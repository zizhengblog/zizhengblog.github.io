---
layout: post
title: "cocoaPods（二）：使用"
date: 2018-05-21
description: "cocoaPods（二）：使用"
tag: CocoaPods
--- 



- [参考文章：Cocoapods三重奏 （一）安装和使用](hhttps://www.jianshu.com/p/430a78995556)


## 目录
* [基本指令介绍](#content1)
* [三方库集成](#content2)
* [三方库删除](#content3)





<!-- ************************************************ -->
## <a id="content1">基本指令介绍</a> 

- [参考文章：https://www.cnblogs.com/chzheng/p/5949353.html](https://www.cnblogs.com/chzheng/p/5949353.html)

**一、版本查看及帮助**


```
#1 查看版本
pod --version


#2 查看一级指令的帮助
pod  --help  


#3 查看子指令的帮助
pod repo -- help
```

**二、索引库相关**


```
#0 初始化环境，下载索引仓库到本地，一般首次安装时执行
pod setup

#1 添加镜像索引
pod repo add master http://git.oschina.net/akuandev/Specs.git


#2 移除本地索引仓库master
pod repo remove master 


#3 更新仓库
pod repo update

#4 更新特定仓库
pod repo update SDWebImage
```

**三、工程相关**

```
#1 初始化Podfile文件
pod init


#2 搜索第三方库
pod search SDWebImage


#3 集成第三方库
pod install


#4 集成第三方库(不更新本地仓库)
pod install  --no-repo-update   //Skip running `pod repo update`
#注：执行install指令时不会执行pod repo unpate 即不会更新本地仓库


#5 更新集成的第三方库，不会受Podfile.lock文件的限制
pod update
```


<!-- ************************************************ -->
## <a id="content2">三方库集成</a> 


**一、初始化podFile文件**


```
#1 cd到工程的根目录下,执行下面指令
#  该指令执行后会在当前目录下生成Podfile文件
pod init 


#2 查看Podfile文件
cat Podfile
```



**二、编辑Podfile文件**

1、多个target中使用相同的Pods依赖库    
比如，名称为CocoaPodsTest的target和Second的target都需要使用Reachability、SBJson、AFNetworking三个Pods依赖库     
可以使用link_with关键字来实现，将Podfile写成如下方式：

```
link_with 'CocoaPodsTest', 'Second'
platform :ios, '7.0'
pod 'Reachability',  '~> 3.0.0'
pod 'SBJson’, ‘~> 4.0.0'
pod 'AFNetworking', '~> 2.0'
```


2、不同的target使用完全不同的Pods依赖库      
CocoaPodsTest这个target使用的是Reachability、SBJson、AFNetworking三个依赖库    
Second这个target只需要使用OpenUDID这一个依赖库     
这时可以使用target关键字，Podfile的描述方式如下,以do/end 开始和结尾

```
target :'CocoaPodsTest' do
    platform :ios
    pod 'Reachability',  '~> 3.0.0'
    pod 'SBJson', '~> 4.0.0'
    platform :ios, '7.0' #要求第三方库支持的iOS最低版本
    pod 'AFNetworking', '~> 2.0'
end

target :'Second' do
    pod 'OpenUDID', '~> 1.0.0'
end
```



3、使用现有xcworkspace集成第三方库
- [参考文章：https://www.jianshu.com/p/e3cfae830985](https://www.jianshu.com/p/e3cfae830985)

```
workspace 'MyWorkspace.xcworkspace' //workspace文件名
project 'MyApp2/MyApp2.xcodeproj' //主工程路径

target 'MyApp2' do
    platform :ios, '8.0'
    project 'MyApp2/MyApp2.xcodeproj' //工程路径
    pod 'Masonry', '~> 1.0.2'
end

target 'MyApp1' do
    platform :ios, '8.0'
    project 'MyApp1/MyApp1.xcodeproj' //工程路径
    pod 'Masonry', '~> 1.0.2'
    pod 'AFNetworking', '~> 3.1.0'
end
```


4、Podfile中指定类库版本的含义
```
pod 'AFNetworking'              //不显式指定依赖库版本，表示每次都获取最新版本
pod 'AFNetworking', '2.0'       //只使用2.0版本
pod 'AFNetworking', '> 2.0'     //使用高于2.0的版本
pod 'AFNetworking', '>= 2.0'    //使用大于或等于2.0的版本
pod 'AFNetworking', '< 2.0'     //使用小于2.0的版本
pod 'AFNetworking', '<= 2.0'    //使用小于或等于2.0的版本
pod 'AFNetworking', '~> 0.1.2'  //使用大于等于0.1.2但小于0.2的版本
pod 'AFNetworking', '~>0.1'     //使用大于等于0.1但小于1.0的版本
pod 'AFNetworking', '~>0'       //高于0的版本，写这个限制和什么都不写是一个效果，都表示使用最新版本
```

5、use_frameworks!
```
不用 use_frameworks! 时 static libraries   方式 -> 生成.a文件    
使用 use_frameworks! 时 dynamic frameworks 方式 -> 生成.framework   
```


**三、安装三方库**


```
#1 在添加之前可以先搜索下要添加的三方库
pod search SDWebImage


#2 如果搜索失败可尝试删除search_index.json文件,然后再执行pod search
rm ~/Library/Caches/CocoaPods/search_index.json


#3 安装第三方库 
#  在工程的根目录下执行该指令集成第三方库
pod install 


#4 重新打开工程
#  关闭xcode,在当前工程目录下打开 pod**.xcworkspace
```

关于pod install 的说明

1 执行结果     
执行完会在当前文件夹下生成 Podfile.lock文件 Pods文件夹 pod**.xcworkspace文件        
其中Podfile.lock 是在pod install时根据Podfile文件 锁定的安装版本          
再次执行pod install时会保持这个锁定的版本不变，目的是保证不同的开发人员使用的三方库版本相同     
所以提交共享库时一定要提交Podfile.lock文件         


2 使用pod install命令安装框架后的大致过程：         
2.1 分析依赖       
该步骤会分析Podfile,查看不同类库之间的依赖情况。如果有多个类库依赖于同一个类库，但是依赖于不同的版本，那么cocoaPods会自动设置一个兼容的版本。     
2.2 下载依赖:        
根据分析依赖的结果，下载指定版本的类库到本地项目中。       
2.3 生成Pods项目：    
创建一个Pods项目专门用来编译和管理第三方框架,CocoaPods会将所需的框架，库等内容添加到项目中，并且进行相应的配置。       
2.4 整合Pods项目：    
将Pods和项目整合到一个工作空间中，并且设置文件链接。    
    




**四、更新三方库**


```
#1 更新所有三方库
pod update


#2 更新指定三方库
pod update LCCommon
```



**五、pod install与pod update的区别**

- [参考文章：https://blog.csdn.net/jhope/article/details/81535586](https://blog.csdn.net/jhope/article/details/81535586)

```
pod install
第一次执行会生成Podfile.lock文件 内会锁定的下载的版本 当再次执行pod install时 会按照Podfile.lock内锁定的版本下载
podfile内新增加三方库时 使用pod install 不影响Podfile.lock内锁定的版本


pod update 
不受Podfile.lock文件影响 只按podfile文件内规定的版本进行下载三方库     
pod update 更新所有第三方库     
pod update AFNetworking 更新指定的第三方库      


pod outdated
每当你运行pod outdated命令时，CocoaPods会列出所有在Podfile.lock中的有新版本的pod库。
这意味着当你对这些pod使用pod update PODNAME时，他们会更新
只要新版本仍然遵守你在Podfile中做的类似于pod 'MyPod', '~>x.y'这样的限制

```

<!-- ************************************************ -->
## <a id="content3">三方库删除</a> 

删除工程文件夹下的Podfile、Podfile.lock及Pods文件夹

删除xcworkspace文件

使用xcodeproj文件打开工程，删除Frameworks组下的 Pods.xcconfig及libPods.a引用

在工程设置中的Build Phases下删除pods相关内容

<img src="/images/tools/tool5.png" alt="img">




----------
>  行者常至，为者常成！



