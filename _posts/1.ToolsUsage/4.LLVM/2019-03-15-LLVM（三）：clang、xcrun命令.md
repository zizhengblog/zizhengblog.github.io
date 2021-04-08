---
layout: post
title: "LLVM（三）：clang、xcrun命令"
date: 2019-03-15 
description: "LLVM（三）：clang、xcrun命令"
tag: LLVM
--- 



- [参考文章：iOS编译命令 clang xcrun](https://www.jianshu.com/p/80240af0bac6)



## 目录
* [clang命令](#content1)
* [xcrun命令](#content2)


<!-- ************************************************ -->
## <a id="content1"></a> clang命令

编译成可执行文件
```
clang main.m -o main -framework Foundation
```

将OC代码转换成C++代码
```
clang -rewrite-objc main.m
```

__weak、__strong等修饰符报错,这是因为__weak需要运行时objc的支持，需要指定运行时环境
```
clang -rewrite-objc -fobjc-arc -fobjc-runtime=ios-8.0.0 main.m
```

引入了UIKit框架
```
clang -rewrite-objc -fobjc-arc -fobjc-runtime=ios-8.0.0 -isysroot /Applications/Xcode.app/Contents/Developer/Platforms/iPhoneSimulator.platform/Developer/SDKs/iPhoneSimulator12.1.sdk main.m
```



<!-- ************************************************ -->
## <a id="content2"></a> xcrun命令

其实，xcode安装的时候顺带安装了xcrun命令，xcrun命令在clang的基础上进行了一些封装，要更好用一些。


将OC代码转换成C++代码
```
##### 在模拟器下编译
xcrun -sdk iphonesimulator clang -rewrite-objc main.m

#在真机下编译
xcrun -sdk iphoneos clang -rewrite-objc main.m
```

指定架构
```
xcrun -sdk iphoneos clang -arch arm64 -rewrite-objc main.m
```

运行时支持
```
xcrun -sdk iphoneos clang -arch arm64 -rewrite-objc -fobjc-arc -fobjc-runtime=ios-9.0.0 main.m
```

有时候我们在本机安装了多个Xcode，可以指定xcrun使用不同的Xcode对应的SDK
```
xcode-select -s /Applications/Xcode9.4.1.app
```

列出当前xcrun对应的SDK的版本的命令
```
xcodebuild -showsdks
```



----------
>  行者常至，为者常成！



