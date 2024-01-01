---
layout: post
title: "CFBundleShortVersionString 与 CFBundleVersion"
date: 2018-02-05
tag: Objective-C
---


[参考:info.plist key 说明](https://developer.apple.com/library/archive/documentation/General/Reference/InfoPlistKeyReference/Articles/CoreFoundationKeys.html#//apple_ref/doc/uid/20001431-111349)      



## 目录
* [CFBundleShortVersionString](#content1)
* [CFBundleVersion](#content2)


<!-- ************************************************ -->
## <a id="content1">CFBundleShortVersionString</a>

CFBundleShortVersionString

CFBundleShortVersionString (String - iOS, macOS) specifies the release version number of the bundle, which identifies a released iteration of the app.

The release version number is a string composed of three period-separated integers. The first integer represents major revision to the app, such as a revision that implements new features or major changes. The second integer denotes a revision that implements less prominent features. The third integer represents a maintenance release revision.

The value for this key differs from the value for CFBundleVersion, which identifies an iteration (released or unreleased) of the app.

This key can be localized by including it in your InfoPlist.strings files.


CFBundleShortVersionString

CFBundleShortVersionString (String - iOS, macOS)指定bundle的发布版本号，它标识应用的一个已发布迭代。


**一 CFBundleShortVersionString**

发布版本号是一个由三个用句号分隔的整数组成的字符串。

第一个整数表示应用程序的主要修订，例如实现新特性或主要更改的修订。

第二个整数表示实现不太突出的特性的修订。

第三个整数表示维护发布版本。

这个键的值与CFBundleVersion的值不同，CFBundleVersion标识应用程序的迭代(已发布或未发布)。

这个键可以通过在InfoPlist中包含它进行本地化。字符串的文件。



<!-- ************************************************ -->
## <a id="content2">CFBundleVersion</a>

CFBundleVersion

CFBundleVersion (String - iOS, macOS) specifies the build version number of the bundle, which identifies an iteration (released or unreleased) of the bundle.

The build version number should be a string comprised of three non-negative, period-separated integers with the first integer being greater than zero—for example, 3.1.2. The string should only contain numeric (0-9) and period (.) characters. Leading zeros are truncated from each integer and will be ignored (that is, 1.02.3 is equivalent to 1.2.3). The meaning of each element is as follows:

* The first number represents the most recent major release and is limited to a maximum length of four digits.

* The second number represents the most recent significant revision and is limited to a maximum length of two digits.

* The third number represents the most recent minor bug fix and is limited to a maximum length of two digits.

If the value of the third number is 0, you can omit it and the second period.

While developing a new version of your app, you can include a suffix after the number that is being updated; for example 3.1.3a1. The character in the suffix represents the stage of development for the new version. For example, you can represent development, alpha, beta, and final candidate, by d, a, b, and fc. The final number in the suffix is the build version, which cannot be 0 and cannot exceed 255. When you release the new version of your app, remove the suffix.


**一 CFBundleVersion**

CFBundleVersion (String - iOS, macOS)指定bundle的构建版本号，它标识bundle的迭代(已发布或未发布)。



构建版本号应该是一个字符串，由三个非负的、以句点分隔的整数组成，其中第一个整数大于零，例如3.1.2。字符串只能包含数字(0-9)和句点(.)。从每个整数截断前导零并将被忽略(也就是说，1.02.3等同于1.2.3)。每个元素的含义如下:

第一个数字表示最近的主要版本，最大长度为4位。

第二个数字表示最近的重要修订，最大长度为两位数。

第三个数字表示最近的小错误修复，最大长度为两位数。

如果第三个数字的值为0，可以省略它和第二个周期。

在开发新版本的应用时，你可以在要更新的数字后面加上后缀;例如3.1.3a1。后缀中的字符表示新版本的开发阶段。例如，您可以用d、a、b和fc表示development、alpha、beta和final candidate。后缀中的最后一个数字是构建版本，不能为0，也不能超过255。当你发布新版本的应用程序时，删除后缀。


**二 xcode中的官方文档的解释**

标识包迭代的构建版本。

讨论

该键是一个机器可读的字符串，由1到3个以句点分隔的整数组成，例如10.14.1。字符串只能包含数字字符(0-9)和句点。

每个整数表示构建版本信息，格式为[Major].[Minor].[Patch]:

Major:主要的修订号。

Minor:次要版本号。

补丁:维护版本号。

你可以包含更多的整数，但是系统会忽略它们。

您还可以通过只使用一个或两个整数来简化构建版本，其中格式中缺少的整数将被解释为零。例如，0指定0.0.0,10指定10.0.0,10.5指定10.5.0。

该密钥是App Store所需要的，并在整个系统中用于识别构建版本。对于macOS应用程序，在发布构建之前增加构建版本。

所以CFBundleVersion = 210 代表的是 210.0.0




----------
>  行者常至，为者常成！


