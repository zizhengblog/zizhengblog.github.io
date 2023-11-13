---
layout: post
title: "cocoaPods（八）：报错和警告"
date: "2023-10-05"
description: ""
tag: CocoaPods
--- 




## 目录
* [repos目录](#content1)


<!-- ************************************************ -->
## <a id="content1">报错和警告</a> 

#### 一、找不到库的podspec文件

```
21:51:20 › pod install
Analyzing dependencies
[!] Unable to find a specification for `MJExtension` depended upon by `XYTestModule`

You have either:
 * out-of-date source repos which you can update with `pod repo update` or with `pod install --repo-update`.
 * mistyped the name or version.
 * not added the source repo that hosts the Podspec to your Podfile.
```
在podfile文件内如果不指定源，会使用默认的源：https://cdn.cocoapods.org/<br>
但当我们指定了一个源的时候就会使用我们指定的源,这时候我们的私有源里是没有MJExtension所以报错了<br>

解决办法：手动添加上默认源
```
source 'https://cdn.cocoapods.org/'
source 'https://e.coding.net/lxy911/xyappmobilespec/XYAppMobileSpec.git'
```

#### 二、私有源和默认源同时包含了AFNetworking

```
21:55:52 › pod install
Analyzing dependencies
Downloading dependencies
Installing AFNetworking (4.0.1)
Installing FMDB (2.7.5)
Installing MJExtension (3.4.1)
Installing MJRefresh (3.7.6)
Installing Masonry (1.1.0)
Installing PLCrashReporter (1.11.1)
Installing ReactiveCocoa (2.3)
Installing SDWebImage (5.18.3)
Installing SDWebImageWebPCoder (0.14.2)
Installing XYFoundation (0.1.1)
Installing XYJumpCenter (0.1.0)
Installing XYTestModule (0.1.2)
Installing XYUIKit (0.1.0)
Installing YYCategories (1.0.4)
Installing libwebp (1.3.2)
Generating Pods project
Integrating client project
Pod installation complete! There is 1 dependency from the Podfile and 15 total pods installed.

[!] Found multiple specifications for `AFNetworking (3.1.0)`:
- /Users/lxy/.cocoapods/repos/trunk/Specs/a/7/5/AFNetworking/3.1.0/AFNetworking.podspec.json
- /Users/lxy/.cocoapods/repos/XYAppMobileSpec/AFNetworking/3.1.0/AFNetworking.podspec

[!] AFNetworking has been deprecated in favor of Alamofire
```
这个警告会在首次执行pod install时提醒，第二次执行时podfile.lock内已经有AFN选定的版本了不会再提醒<br>
由于没有指定AFNetworking的版本，所以默认使用最新的版本<br>

解决办法：<br>
要想使用私有源的版本，请指定3.1.0，这个时候会优先去私有源里去找<br>

#### 三、pod install 后出现了大量的黄色警告

```
[!] [Xcodeproj] Generated duplicate UUIDs:

PBXBuildFile -- Pods.xcodeproj/targets/
buildConfigurationList:buildConfigurations:baseConfigurationReference:|,buildSettings:|,displayName:|,isa:|,
name:|,,baseConfigurationReference:|,buildSettings:|,displayName:|,isa:|,name:|,,
defaultConfigurationIsVisible:0,defaultConfigurationName:Release,displayName:ConfigurationList,
isa:XCConfigurationList,,buildPhases:buildActionMask:2147483647,displayName:Headers,
files:|,|,|,|,|,|,|,|,|,|,|,|,|,|,|,|,|,|,|,|,|,|,|,|,|,|,|,|,|,|,|,|,|,|,|,|,|,|,|,|,|,|,|,|,|,|,|,|,|,|,|,|
,|,|,|,|,|,|,|,|,|,|,|,|,|,|,|,|,|,|,|,|,|,|,|,|,|,|,|,|,|,|,|,|,|,|,|,|,|,|,|,|,|,|,|,|,|,|,|,|,|,|,|,|,|,|,
|,|,|,|,|,|,|,|,|,|,|,|,|,|,|,|,|,isa:PBXHeadersBuildPhase,runOnlyForDeploymentPostprocessing:0,,buildActionMask:2147483647,displayName:Sources,files
...
```
出现这个问题的原因是podspec文件内的sourfile写的有问题导致的
```
// 本应该写成
s.source_files = 'Classes/**/*'

// 实际写成了
s.source_files = 'XYTestModule/Classes/**/*'
```
不同的写法会影响pod工程下文件的目录层级，但具体报错的原因还是没明白？

----------
>  行者常至，为者常成！



