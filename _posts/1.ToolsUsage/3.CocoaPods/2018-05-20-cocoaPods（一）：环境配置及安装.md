---
layout: post
title: "cocoaPods（一）：环境配置及安装"
date: 2018-05-20 
description: "cocoaPods（一）：环境配置及安装"
tag: CocoaPods
--- 


- [参考文章：Cocoapods官网](https://cocoapods.org/)
- [参考文章：Cocoapods三重奏 （一）安装和使用](https://www.jianshu.com/p/430a78995556)


## 目录
* [环境配置](#content1)
* [cocoaPods](#content2)




<!-- ************************************************ -->
## <a id="content1">环境配置</a> 

RVM(ruby管理器) -> ruby(ruby环境) -> gem(包管理器)

**一、安装RVM**

什么是RVM呢？RVM的全称是Ruby Version Manager，翻译过来就是Ruby版本管理器.    
RVM是一个命令行工具，通过它可以轻松的安装、管理多个Ruby环境        
如果想更多的了解RVM可以通过访问 <a>http://www.rvm.io</a>         
     
```
#1 安装指令
curl -L https://get.rvm.io | bash -s stable

#2 查看是否安装成功
rvm -v
```

**二、用RVM来安装Ruby环境**

ruby 是一种语言，是某些软件包代码的执行环境，MAC系统有一个默认的ruby版本，ruby可以安装多个不同的版本


```
#1 查询并且列出已知的ruby版本
rvm list known

#2 查询已经安装的ruby版本
rvm list

#3 安装ruby
rvm install 版本号

#4 查看当前默认版本
ruby -v

#5 设置Ruby默认版本
rvm 版本号 --default

#6 卸载ruby
rvm remove 版本号
```


**三、RubyGems**

RubyGems(gem) 是 Ruby 的一个包管理器，它提供一个分发 Ruby 程序和库的标准格式，还提供一个管理程序包安装的工具。      
gem命令用于构建、上传、下载以及安装Gem包。


```
#1 版本查看
gem -v

#2 列出可用的gem包
gem list --remote

#3 列出已安装的gem包
gem list --local

#4 安装gem管理的包
gem install 包名

#5 更新所有程序包
gem update

#6 更新某一程序包
gem update cocoapods

#7 卸载已安装的gem包
gem uninstall 包名
```

gem太老可能会有问题，所以在安装gem包之前最好先更新升级gem。     
由于国内网络原因（你懂的），导致 rubygems.org 存放在 Amazon S3 上面的资源文件间歇性连接失败。    
所以你会遇到 gem install rack 或 bundle install 的时候半天没有响应，具体可以用 gem install rails -V 来查看执行过程。    
因此我们可以将它修改为国内的下载源: https://gems.ruby-china.com      


```
#1 查看当前源
gem sources -l
*** CURRENT SOURCES ***

https://rubygems.org/


#2 移除 https://rubygems.org/，并添加国内下载源      
gem sources --remove https://rubygems.org/


#3 添加国内源 https://ruby.taobao.org/ (淘宝的源已不再更新，不建议继续使用)     
gem sources -a https://gems.ruby-china.com/


#4 请确保只有 gems.ruby-china.com
gem sources -l
*** CURRENT SOURCES ***

https://gems.ruby-china.com/


#5 更新gem本身
sudo gem update --system
```


<!-- ************************************************ -->
## <a id="content2">cocoaPods</a> 


**一、cocoaPods介绍**

cocoaPods项目开始于2011年8月12日,其源码在Github上管理，且持续保持活跃更新。    
简单来说，cocoaPods是一个负责管理iOS项目中第三方开源库的工具。       
<span style="color:red">我们可以通过Podfile文件添加并管理第三方库。</span>         
<span style="color:red">通过pod install命令自动将这些第三方开源库的源码下载下来，并且为我的工程设置好相应的系统依赖和编译参数。</span>        
相对于手动管理（手动拖拽和删除）第三方库来说确实方便了很多，节省时间去关心第三方库的使用。      



**二、cocoaPods的安装**

cocoaPods的安装需要使用ruby的gem，Mac下已经自带了ruby，只要使用ruby的gem命令就可以安装了。

```
#1 安装cocoaPods
# 目前执行该条指令会报错，是因为文件权限问题，Mac系统目前不提倡在系统的bin目录 /usr/bin 下安装指令
# ERROR:  While executing gem ... (Gem::FilePermissionError)
sudo gem install cocoapods


#2 所以使用下面的指令，将pod指令安装到本地的bin目录下：
sudo gem install -n /usr/local/bin cocoapods


#3 查看版本，如果能查看到具体的版本，cocoapods安装成功。
pod --version 
```

**三、设置pod索引库**


```
#1 设置索引库
#这步其实是Cocoapods在将它的镜像索引下载到 ~/.cocoapods/repos目录下。       
#CocoaPods的所有项目的镜像索引Podspec文件都托管在https://github.com/CocoaPods/Specs。       
#Podspec文件是我们使用cocoaPods找到第三方库源码的索引文件，每个第三方库都有一个。      
#第一次执行pod setup时，CocoaPods会将这些podspec索引文件更新到本地的~/.cocoapods/repos/master目录下.
#这个索引文件比较大，所以第一次更新时非常慢。
pod setup


#2 为了提高下载速度可以使用下面两种方法：  
#第一种方法：
#可以将文件托管地址从github替换为国内oschina这样会快很多。            
#如下操作就可以将github替换为国内oschinapod      
#移除本地仓库master
pod repo remove master  


#3 或者可以手动删除，路径~/.cocoapods/repos/master


#4 添加镜像索引
pod repo add master http://git.oschina.net/akuandev/Specs.git 
 

#5 第二种方法：
#如果pod指令不好用或长时间没反应可以cd到 `~/.cocoapods/repos` 目录下 通过git指令clone master仓库
git clone https://github.com/CocoaPods/Specs.git master


#6 第三种方法：
#将其它电脑 ~/.cocoapods/repos/ 下的master文件夹拷贝到自己电脑     
#参考文章：https://www.jianshu.com/p/0909d2c126a5   


#7 更新pod仓库
pod repo update


#8 或者手动删除后重新设置
pod repo remove master
pod setup
```


**四、卸载cocoaPods**

- [参考文章：https://blog.csdn.net/qq_32666701/article/details/80607646](https://blog.csdn.net/qq_32666701/article/details/80607646)

```
#1 卸载cocoapods
sudo gem uninstall cocoapods


#2 查看本地安装过的cocoapods相关文件
gem list --local | grep cocoapods


#3 然后使用命令逐个删除
sudo gem uninstall cocoapods-core


#4 如果怕删不干净有残留的话可以找到 .cocopods 文件（隐藏文件）删掉就好
```



**五、切换不同版本cocoapods**

一种方案是安装不同版本的ruby，在不同的ruby下安装特定的cocoapods版本，通过切换ruby环境切换不同的cocoapods版本。

Cocoapods 版本        
<a>https://www.cnblogs.com/willbin/p/10947534.html</a>


多个pod版本切换       
<a>https://www.jianshu.com/p/78ae1c6749a0</a>


CocoaPods切换不同版本        
<a>https://www.jianshu.com/p/68d3975ab10a</a>



----------
>  行者常至，为者常成！



