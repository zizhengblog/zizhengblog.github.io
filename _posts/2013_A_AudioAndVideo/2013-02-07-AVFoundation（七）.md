---
layout: post
title: "7、AV Foundation之视频捕捉-拍摄静态图片"
date: 2013-02-07
description: ""
tag: 音视频
---






- [文章摘抄自：https://www.jianshu.com/p/3d5565f65db3，用于记录学习](https://www.jianshu.com/p/3d5565f65db3)





## 目录

* [前言](#content1)
* [捕捉静态图片](#content2)
* [使用ASSets Library框架](#content3)







<!-- ************************************************ -->
## <a id="content1"></a>前言

回顾一下，我们在setupSession：方法中，将一个AVCaptureStillImageOutput实例添加到捕捉会话。这个类是AVCaptureOutput 的子类，用于捕捉静态图片。


创建一个会话并添加捕捉设备输入&捕捉输出时，会话自动建立输入和输出的连接，按需选择信号流线路。访问这些连接在一些情况下是非常实用的功能，因为可以让开发者更好地对发送到输出端的数据进行控制。
```
AVCaptureConnection *connection = // Active video capture connection

id completionHandler = ^(CMSampleBufferRef buffer,NSError *error){

     //Handle image capture 

}

[imageOutput captureStillImageAsynchronouslyFromConnection:connection completionHandler:completionHandler];

```

<!-- ************************************************ -->
## <a id="content2"></a>捕捉静态图片

```
#pragma mark - Image Capture Methods 拍摄静态图片
/*
    AVCaptureStillImageOutput 是AVCaptureOutput的子类。用于捕捉图片
 */
- (void)captureStillImage {
    
    //获取连接
    AVCaptureConnection *connection = [self.imageOutput connectionWithMediaType:AVMediaTypeVideo];
    
    //程序只支持纵向，但是如果用户横向拍照时，需要调整结果照片的方向
    //判断是否支持设置视频方向
    if (connection.isVideoOrientationSupported) {
        
        //获取方向值
        connection.videoOrientation = [self currentVideoOrientation];
    }
    
    //定义一个handler 块，会返回1个图片的NSData数据
    id handler = ^(CMSampleBufferRef sampleBuffer,NSError *error)
                {
                    if (sampleBuffer != NULL) {
                        NSData *imageData = [AVCaptureStillImageOutput jpegStillImageNSDataRepresentation:sampleBuffer];
                        UIImage *image = [[UIImage alloc]initWithData:imageData];
                        
                        //重点：捕捉图片成功后，将图片传递出去
                        [self writeImageToAssetsLibrary:image];
                    }else
                    {
                        NSLog(@"NULL sampleBuffer:%@",[error localizedDescription]);
                    }
                        
                };
    
    //捕捉静态图片
    [self.imageOutput captureStillImageAsynchronouslyFromConnection:connection completionHandler:handler];
    
    
    
}

//获取方向值
- (AVCaptureVideoOrientation)currentVideoOrientation {
    
    AVCaptureVideoOrientation orientation;
    
    //获取UIDevice 的 orientation
    switch ([UIDevice currentDevice].orientation) {
        case UIDeviceOrientationPortrait:
            orientation = AVCaptureVideoOrientationPortrait;
            break;
        case UIDeviceOrientationLandscapeRight:
            orientation = AVCaptureVideoOrientationLandscapeLeft;
            break;
        case UIDeviceOrientationPortraitUpsideDown:
            orientation = AVCaptureVideoOrientationPortraitUpsideDown;
            break;
        default:
            orientation = AVCaptureVideoOrientationLandscapeRight;
            break;
    }
    
    return orientation;

    return 0;
}
```


<!-- ************************************************ -->
## <a id="content3"></a>使用ASSets Library框架

ASSets Library 框架可以让开发者通过编程方式访问iOS Photos 应用程序所管理的用户相册&视频库。这个框架的核心类是：ALAssetsLibrary.ALAssetsLibrary类定义了于用户库进行交互的接口。该对象具有多个“写入”方法，可以让开发者将照片&视频写入到自己的库中。

当然涉及到用户隐私，同样需要在plist中修改，还需要在用户的允许才能访问相册。

```
/*
    Assets Library 框架 
    用来让开发者通过代码方式访问iOS photo
    注意：会访问到相册，需要修改plist 权限。否则会导致项目崩溃
 */

- (void)writeImageToAssetsLibrary:(UIImage *)image {

    //创建ALAssetsLibrary  实例
    ALAssetsLibrary *library = [[ALAssetsLibrary alloc]init];
    
    //参数1:图片（参数为CGImageRef 所以image.CGImage）
    //参数2:方向参数 转为NSUInteger
    //参数3:写入成功、失败处理
    [library writeImageToSavedPhotosAlbum:image.CGImage
                             orientation:(NSUInteger)image.imageOrientation
                         completionBlock:^(NSURL *assetURL, NSError *error) {
                             //成功后，发送捕捉图片通知。用于绘制程序的左下角的缩略图
                             if (!error)
                             {
                                 [self postThumbnailNotifification:image];
                             }else
                             {
                                 //失败打印错误信息
                                 id message = [error localizedDescription];
                                 NSLog(@"%@",message);
                             }
                         }];
}

//发送缩略图通知
- (void)postThumbnailNotifification:(UIImage *)image {
    
    //回到主队列
    dispatch_async(dispatch_get_main_queue(), ^{
        //发送请求
        NSNotificationCenter *nc = [NSNotificationCenter defaultCenter];
        [nc postNotificationName:THThumbnailCreatedNotification object:image];
    });
}

```






----------
>  行者常至，为者常成！


