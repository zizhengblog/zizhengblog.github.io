---
layout: post
title: "cocoaPods（七）：repos目录"
date: "2023-10-04"
description: ""
tag: CocoaPods
--- 




## 目录
* [repos目录](#content1)


<!-- ************************************************ -->
## <a id="content1">repos目录</a> 

#### 一、repos目录介绍

repos目录位于~/.cocoapods/repos下

存放了各个三方库的源仓库，其中trunk是官方的源仓库，其它的是私有源仓库

```shell
$ ls ~/.cocoapods/repos

XYAppMobileSpec                    trunk
```


#### 二、trunk目录介绍

trunk目录内的文件和文件夹 

**all_pods.txt 文件**：这个文件列出了所有可用库的名称。它通常包含了所有库的清单，供CocoaPods使用。

**Specs 目录**：这个目录下存放了所有三方库的podspec文件,<span style="color:red;font-weight:bold">存放规则是按三方库名称的md5值进行排列的</span>

```shell
$ echo -n "AFNetworking" | md5

a75d452377f3996bdc4b623a5df25820
```
在目录trunk/a/7/5 目录下就会找到AFNetworking这个三方库的podspec文件



----------
>  行者常至，为者常成！



