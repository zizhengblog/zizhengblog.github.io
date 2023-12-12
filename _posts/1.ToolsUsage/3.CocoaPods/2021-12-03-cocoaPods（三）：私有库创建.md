---
layout: post
title: "cocoaPods（三）：私有库创建"
date: 2021-12-03
description: "cocoaPods（三）：私有库创建"
tag: CocoaPods
--- 





## 目录
* [私有pod库创建](#content1)
* [私有pod库索引库创建](#content2)
* [私有库使用](#content3)
* [私有库更新](#content4)
* [spec文件](#content5)





<!-- ************************************************ -->
## <a id="content1">私有pod库创建</a>

- [参考文章：https://www.jianshu.com/p/36953a48937d](https://www.jianshu.com/p/36953a48937d)
- [参考文章：https://cloud.tencent.com/developer/article/1336311](https://cloud.tencent.com/developer/article/1336311)



  
```   
#1 github创建一个仓库 用来存储工程文件
https://github.com/JiangHuHiKe/LCCommon.git


#2 创建pod私有库的项目工程      
在合适的目录下执行 pod lib create 命令 按提示输入需要的内容
创建名字叫LCCommon的私有库项目
pod lib create LCCommon   


#3 添加文件并更新   
在目录 ../LCCommon/LCCommon/Classes下 删除"ReplaceMe.m"文件
在目录 ../LCCommon/LCCommon/Classes下 添加LCCategory文件夹，内包含UIColor+Category.h UIColor+Category.m文件
在目录 ../LCCommon/Example下执行 pod install  更新Example项目中的pod 出现提示成功字样 该步完成


#4 修改podspec文件并验证   
打开../LCCommon/Example 中的.workspace文件 打开工程    
找到LCCommon.podspec文件 进行修改   
在目录 ../LCCommon下执行pod lib lint 进行验证 出现成功字样 该步完成
   
    
#5 将本地项目文件上传到远程私有库中 并 校验spec
在目录../LCCommon下执行：
$ git remote add origin https://github.com/JiangHuHiKe/LCCommon.git
$ git add .
$ git commit -m "Initial LCCommon"
$ git push -u origin master
$ git tag 0.1.0     //tag 值要和podspec中的version一致
$ git push --tags   //推送tag到服务器上

在目录../LCCommon下执行：
pod spec lint 
校验spec 出现成功提示该步完成，出现错误重新执行该步骤
```
    
 
 
<!-- ************************************************ -->
## <a id="content2">私有pod库索引库创建</a> 


```
#1 github创建一个仓库 用来作为索引库
https://github.com/JiangHuHiKe/LCCommonSpec.git


#2 将索引库克隆到 ~/.cocoaPods/repos目录下
在目录../LCCommon下执行：
pod repo add LCCommonSpec https://github.com/JiangHuHiKe/LCCommonSpec.git


#3 建立关联
在目录../LCCommon下执行：
pod repo push LCCommonSpec LCCommon.podspec 
```



<!-- ************************************************ -->
## <a id="content3">私有库使用</a> 


```
#1 开始集成前可先搜索
pod search LCCommon


#2 更新索引库
pod repo update LCCommonSpec


#3 新建工程的根目录下执行
pod init


#4 修改Podfile文件如下
source 'https://github.com/CocoaPods/Specs.git'
source 'https://github.com/JiangHuHiKe/LCCommonSpec.git'
target 'podUsageTest' do
#use_frameworks!
platform :ios, '9.0'
pod 'MJExtension'
pod 'LCCommon'
end


#5 集成
pod install
```



<!-- ************************************************ -->
## <a id="content4">私有库更新</a> 



```
#1 添加文件并更新
添加完成后
在目录 ../LCCommon/Example下执行 pod install  更新Example项目中的pod 出现提示成功字样 该步完成

    
#2 修改 LCCommon.podspec文件
找到LCCommon.podspec文件 进行修改 s.version的值
在目录 ../LCCommon下执行pod lib lint 进行验证 出现成功字样 该步完成


#3 推送到远程并更新tag
git add .
git commit -m "your message"
git push
git tag 0.1.1   //tag值要与s.version一致
git push --tags //

在目录../LCCommon下执行：
pod spec lint //有警告的话可以加上 --allow-warnings 来忽略警告
校验spec 出现成功提示该步完成，出现错误重新执行该步骤


#4 更新本地索引库
pod repo push LCCommonSpec LCCommon.podspec

```



<!-- ************************************************ -->
## <a id="content5">spec文件</a> 

**一、文件说明**

- [参考文章：podspec文件介绍](https://www.jianshu.com/p/a23397065e40)

```
name：框架名
version：当前版本（注意，是当前版本，假如你后续更新了新版本，需要修改此处）
summary：简要描述，在pod search ZCPKit的时候会显示该信息。
description：详细描述
homepage：页面链接
license：开源协议
author：作者
source：源码git地址
platform：支持最低ios版本
source_files：源文件（可以包含.h和.m）
public_header_files：头文件(.h文件)
resources：资源文件（配置的文件都会被放到mainBundle中）
resource_bundles：资源文件（配置的文件会放到你自己指定的bundle中）
frameworks：依赖的系统框架
vendored_frameworks：依赖的非系统框架
libraries：依赖的系统库
vendored_libraries：依赖的非系统的静态库
dependency：依赖的三方库
```


**二、子模块**

- [参考文章：https://www.jianshu.com/p/6bb4980a8af6](https://www.jianshu.com/p/6bb4980a8af6)
- [参考文章：https://www.jianshu.com/p/e990bce53431](https://www.jianshu.com/p/e990bce53431)
- [参考文章：http://blog.wtlucky.com/blog/2015/02/26/create-private-podspec/](http://blog.wtlucky.com/blog/2015/02/26/create-private-podspec/)

```
#该行会影响子目录的文档结构 要与subspec配合使用
s.source_files = 'LCCommon/Classes/**/*'

s.subspec 'TestC' do |testc|
testc.source_files = 'Test/TestC/*.{h,m}'
end

s.subspec '目录名字' do |别名-小写字母|
别名.source_files = '文件路径'
end
```


注意点
```
1、如果搜索某个仓库找不到或没有反应
    移除~/.cocoapods/repos/trunk文件夹后尝试
    
    如不行更新本地索引库
    pod repo update
 
    如果还是失败 移除以下文件
    rm ~/Library/Caches/CocoaPods/search_index.json 
    参考文章：
    https://blog.csdn.net/conglin1991/article/details/55096422?utm_source=blogxgwz9
    https://www.jianshu.com/p/9add72f11df6
    
2、理解CocoaPods的Pod Lib Create
    https://www.jianshu.com/p/4685af9dd219
    
3、pod lib lint出错
    https://www.jianshu.com/p/6bc293da596c
    https://blog.csdn.net/adadadadadadad40/article/details/87805144

4、私有库xxx.podspec文件相关
    https://juejin.im/post/5b04e26bf265da0b83371a9c
    
5、执行指定的
    pod 'lklZFWxSDK', :path => '../'
```

----------
>  行者常至，为者常成！



