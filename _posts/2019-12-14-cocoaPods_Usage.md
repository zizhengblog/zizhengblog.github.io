---
layout: post
title: "cocoaPods_Usage"
date: 2019-12-14 
description: "cocoaPods使用"
tag: 工具
--- 

<h6>
  <img src="https://robotkang-1257995526.cos.ap-chengdu.myqcloud.com/icon/copyright.png" alt="copyright" style="display:inline;margin-bottom: -5px;" width="20" height="20"> 版权声明：本文为博主原创文章，未经博主允许不得转载。

  <a target="_blank" href="https://jianghuhike.github.io/191214.html">
  原文地址：https://jianghuhike.github.io/191214.html 
  </a>
</h6>



## 目录
* [ 相关概念](#content0)
* [ 一、安装cocoaPods](#content1)
* [ 二、使用cocoaPods](#content2)
* [ 三、私有pod库](#content3)
* [ 四、注意点](#content4)



## <a id="content0"></a> 相关概念
```
ruby 
是一种动态语言，类似python由一个日本人开发。

gem 
是一种文件组织的包，一般的ruby的很多插件都有由这种各种的包提供
你可以理解为开发ruby时的插件,像C++的库。


CocoaPods
是一个负责管理iOS项目中第三方开源库的工具。
CocoaPods的项目源码在Github上管理。
该项目开始于2011年8月12日，在这两年多的时间里，它持续保持活跃更新。
开发iOS项目不可避免地要使用第三方开源库，CocoaPods的出现使得我们可以节省设置和更新第三方开源库的时间
在我们有了CocoaPods这个工具之后，只需要将用到的第三方开源库放到一个名为Podfile的文件中，然后在命令行执行$ pod install命令。
CocoaPods就会自动将这些第三方开源库的源码下载下来，并且为我的工程设置好相应的系统依赖和编译参数
注：安装cocoaPods的安装需要使用ruby的gem，Mac下已经自带了ruby，只要使用ruby的gem命令就可以安装了。
```


## <a id="content1"></a> 一、安装cocoaPods

### 1.1 安装前环境配置
```
查看ruby版本
ruby -v

查看gem版本
gem -v

查看gem源列表
gem sources -l


gem太老可能会有问题，所以安装之前最好升级一下gem 更新升级gem，国内需要切换源
以下命令添加淘宝的源：（不建议继续使用） gem sources -a https://ruby.taobao.org/

以下命令移除淘宝的源：
gem sources --remove https://ruby.taobao.org/

淘宝的源已经不更新维护了，现在使用ruby-china的源 gem source -a https://gems.ruby-china.org
 
验证是否替换成功 
gem sources -l

确保只显示一个
***CURRENTSOURCES*** 
https://gems.ruby-china.org        
 
更新gem
sudo gem update --system
```

### 1.2  安装cocoaPods及仓库设置
```
安装cocoaPods
sudo gem install -n /usr/local/bin cocoapods


查看版本
pod --version 


设置pod仓库
pod setup
    这步其实是Cocoapods在将它的镜像索引下载到 ~/.cocoapods/repos目录下。
    CocoaPods的所有项目的镜像索引Podspec文件都托管在https://github.com/CocoaPods/Specs。
    Podspec文件是我们使用cocoaPods找到第三方库源码的索引文件，每个第三方库都有一个。
    第一次执行pod setup时，CocoaPods会将这些podspec索引文件更新到本地的~/.cocoapods/目录下，这个索引文件比较大，有100M左右。
    所以第一次更新时非常慢。为了提高下载速度可以使用下面两种方法：
    第一种方法：
    可以将文件托管地址从github替换为国内oschina这样会快很多。如下操作就可以将github替换为国内oschinapod 
    pod repo remove master  //移除本地仓库master删除失败可以手动删除路径~/.cocoapods/repos/master
    pod repo add master http://git.oschina.net/akuandev/Specs.git  //添加镜像索引
    pod repo update    //更新本地repos仓库
    第二种方法：
    https://www.jianshu.com/p/0909d2c126a5
    其它电脑 ~/.cocoapods/repos/ 下的master文件夹拷贝到自己电脑


更新pod仓库
pod repo update
    或者手动删除后重新设置
    pod repo remove master
    pod setup
```


### 1.3 卸载cocoaPods
- [参考文章：https://blog.csdn.net/qq_32666701/article/details/80607646](https://blog.csdn.net/qq_32666701/article/details/80607646)

```
卸载cocoapods
sudo gem uninstall cocoapods

查看本地安装过的cocoapods相关东西
gem list --local | grep cocoapods

然后使用命令逐个删除
sudo gem uninstall cocoapods-core
如果怕删不干净有残留的话可以找到 .cocopods 文件（隐藏文件）删掉就好
```



## <a id="content2"></a> 二、使用cocoaPods

### 2.1 pod常用指令
- [参考文章：https://www.cnblogs.com/chzheng/p/5949353.html](https://www.cnblogs.com/chzheng/p/5949353.html)

```
查看版本
pod --version


查看一级指令的帮助
pod  --help  
查看install下边子指令的帮助
pod install -- help


添加镜像索引 pod repo add master http://git.oschina.net/akuandev/Specs.git
移除本地索引仓库master
pod repo remove master  
设置仓库
pod setup
更新仓库
pod repo update


初始化Podfile文件
pod init
搜索第三方库
pod search SDWebImage
集成第三方库
pod install
pod install  --no-repo-update   //Skip running `pod repo update`
注：执行install指令时不会执行pod repo unpate 即不会更新本地仓库

更新集成的第三方库，不会受Podfile.lock文件的限制
pod update

```

### 2.2 集成第三方库
```
1.cd工程的根目录下初始化podFile文件
    pod init    //该指令执行后会在当前目录下生成Podfile文件

    $ pod init
    $ cat Podfile
    # Uncomment the next line to define a global platform for your project
    # platform :ios, '9.0'

    target 'podTest' do
    # Comment the next line if you don't want to use dynamic frameworks
    use_frameworks!

    # Pods for podTest
    end



2.在Podfile文件添加第三方库如：

    在添加之前可以先搜索下要添加的三方库
    pod search SDWebImage

    编辑Podfile文件
        target 'podTest' do

            use_frameworks!

            #用来设置所有第三方库所支持的iOS最低版本
            platform :ios , '9.0'

            #设置框架的名称和版本号
            pod 'SDWebImage','~>3.7.5' 

            #'>1.0'    可以安装任何高于1.0的版本
            #'>=1.0'   可以安装任何高于或等于1.0的版本
            #'<1.0'    任何低于1.0的版本
            #'<=1.0'   任何低于或等于1.0的版本
            #'~>0.1'   任何高于或等于0.1的版本，但是不包含高于1.0的版本
            #'~>0'     任何版本，相当于不指定版本，默认采用最新版本号
        end


3.安装第三方库 
    pod install //在工程的根目录下执行该指令集成第三方库
    注：执行完会在当前文件夹下生成 Podfile.lock Pods pod**.xcworkspace
    其中Podfile.lock 是在pod install时根据Podfile文件 锁定的安装版本 
    再次执行pod install时会保持这个锁定的版本不变，目的是保证不同的开发人员使用的三方库版本相同，所以提交共享库时一定要提交Podfile.lock文件

    使用pod install命令安装框架后的大致过程：
    01分析依赖:该步骤会分析Podfile,查看不同类库之间的依赖情况。
    如果有多个类库依赖于同一个类库，但是依赖于不同的版本，那么cocoaPods会自动设置一个兼容的版本。
    02 下载依赖:根据分析依赖的结果，下载指定版本的类库到本地项目中。
    03 生成Pods项目：创建一个Pods项目专门用来编译和管理第三方框架，CocoaPods会将所需的框架，库等内容添加到项目中，并且进行相应的配置。
    04 整合Pods项目：将Pods和项目整合到一个工作空间中，并且设置文件链接。
    
4.重新打开工程
关闭xcode,在当前工程目录下打开 pod**.xcworkspace

```


### 2.3 使用现有xcworkspace集成第三方库
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


###  2.4 pod install与pod update的区别
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
每当你运行pod outdated命令时，CocoaPods会列出所有在Podfile.lock中的有新版本的pod库。这意味着当你对这些pod使用pod update PODNAME时，他们会更新（只要新版本仍然遵守你在Podfile中做的类似于pod 'MyPod', '~>x.y'这样的限制
```


### 2.5 master 与 trunk
- [参考文章：https://zhaoxin.pro/15695124897584.html](https://zhaoxin.pro/15695124897584.html)
```
master源与trunk源的选择
```


## <a id="content3"></a> 三、私有pod库
- [参考文章：https://www.jianshu.com/p/36953a48937d](https://www.jianshu.com/p/36953a48937d)
- [参考文章：https://cloud.tencent.com/developer/article/1336311](https://cloud.tencent.com/developer/article/1336311)

### 3.1 创建私有pod库

```
************************************私有pod库项目创建******************************************   

1、github创建一个仓库 用来存储工程文件
    https://github.com/JiangHuHiKe/LCCommon.git

2、创建pod私有库的项目工程
    在合适的目录下执行 pod lib create 命令 按提示输入需要的内容
    $ pod lib create LCCommon   //创建名字叫LCCommon的私有库项目

3、添加文件并更新
    在目录 ../LCCommon/LCCommon/Classes下 删除"ReplaceMe.m"文件
    在目录 ../LCCommon/LCCommon/Classes下 添加LCCategory文件夹，内包含UIColor+Category.h UIColor+Category.m文件
    在目录 ../LCCommon/Example下执行 pod install  更新Example项目中的pod 出现提示成功字样 该步完成

4、修改podspec文件并验证
    打开../LCCommon/Example 中的.workspace文件 打开工程
    找到LCCommon.podspec文件 进行修改
    在目录 ../LCCommon下执行pod lib lint 进行验证 出现成功字样 该步完成
    
5、将本地项目文件上传到远程私有库中 并 校验spec
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
    
************************************私有pod库项目创建完成******************************************   
 
 
 
 
************************************私有pod库索引库创建******************************************   
1、github创建一个仓库 用来作为索引库
    https://github.com/JiangHuHiKe/LCCommonSpec.git

2、将索引库克隆到 ~/.cocoaPods/repos目录下
    在目录../LCCommon下执行：
    pod repo add LCCommonSpec https://github.com/JiangHuHiKe/LCCommonSpec.git

3、建立关联
    在目录../LCCommon下执行：
    pod repo push LCCommonSpec LCCommon.podspec 

************************************私有pod库索引库创建完成******************************************   


```

### 3.2  私有库使用
```
开始集成前可先搜索
pod search LCCommon

更新索引库
pod repo update LCCommonSpec

1、新建工程的根目录下执行：
    pod init

2、修改Podfile文件如下：
    source 'https://github.com/CocoaPods/Specs.git'
    source 'https://github.com/JiangHuHiKe/LCCommonSpec.git'
    target 'podUsageTest' do
    use_frameworks!
    platform :ios, '9.0'
    pod 'MJExtension'
    pod 'LCCommon'
    end

3、集成
    pod install

```

### 3.3 私有库更新
```
1、添加文件并更新
    添加完成后
    在目录 ../LCCommon/Example下执行 pod install  更新Example项目中的pod 出现提示成功字样 该步完成
    
2、修改 LCCommon.podspec文件
    找到LCCommon.podspec文件 进行修改 s.version的值
    在目录 ../LCCommon下执行pod lib lint 进行验证 出现成功字样 该步完成


3、推送到远程并更新tag
    git add .
    git commit -m "your message"
    git push
    git tag 0.1.1   //tag值要与s.version一致
    git push --tags //
    
    在目录../LCCommon下执行：
    pod spec lint //有警告的话可以加上 --allow-warnings 来忽略警告
    校验spec 出现成功提示该步完成，出现错误重新执行该步骤


4、更新本地索引库
    pod repo push LCCommonSpec LCCommon.podspec
```

### 3.4 私有库subspec 子模块
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


## <a id="content4"></a> 四、注意点
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



