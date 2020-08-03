---
layout: post
title: "3、AV Foundation之视频捕捉-创建捕捉控制器"
date: 2019-08-05
description: ""
tag: 音视频
---


<h6>版权声明：本文为博主原创文章，未经博主允许不得转载。</h6>


- [文章摘抄自：https://www.jianshu.com/p/67c32aece62d，用于记录学习](https://www.jianshu.com/p/67c32aece62d)






## 目录

* [THCameraController类中实现](#content1)
* [处理隐私需求](#content2)




<!-- ************************************************ -->
## <a id="content1"></a>THCameraController类中实现

**一、用于配置不同的捕捉设备，同时对捕捉的输出进行控制和交互。**


```
#import <AVFoundation/AVFoundation.h>

extern NSString *const THThumbnailCreatedNotification;

@protocol THCameraControllerDelegate <NSObject>

// 1发生错误事件是，需要在对象委托上调用一些方法来处理
- (void)deviceConfigurationFailedWithError:(NSError *)error;
- (void)mediaCaptureFailedWithError:(NSError *)error;
- (void)assetLibraryWriteFailedWithError:(NSError *)error;
@end

@interface THCameraController : NSObject

@property (weak, nonatomic) id<THCameraControllerDelegate> delegate;
@property (nonatomic, strong, readonly) AVCaptureSession *captureSession;


// 2 用于设置、配置视频捕捉会话
- (BOOL)setupSession:(NSError **)error;
- (void)startSession;
- (void)stopSession;
```


**二、在设置捕捉会话在THCameraController.m**

需要导入系统框架<AVFoundation/AVFoundation.h><AssetsLibrary/AssetsLibrary.h>

设置捕捉会话
```
- (BOOL)setupSession:(NSError **)error {

    
    //创建捕捉会话。AVCaptureSession 是捕捉场景的中心枢纽
    self.captureSession = [[AVCaptureSession alloc]init];
    
    /*
     AVCaptureSessionPresetHigh
     AVCaptureSessionPresetMedium
     AVCaptureSessionPresetLow
     AVCaptureSessionPreset640x480
     AVCaptureSessionPreset1280x720
     AVCaptureSessionPresetPhoto
     */
    //设置图像的分辨率
    self.captureSession.sessionPreset = AVCaptureSessionPresetHigh;
    
    //拿到默认视频捕捉设备 iOS系统返回后置摄像头
    AVCaptureDevice *videoDevice = [AVCaptureDevice defaultDeviceWithMediaType:AVMediaTypeVideo];
    
    //将捕捉设备封装成AVCaptureDeviceInput
    //注意：为会话添加捕捉设备，必须将设备封装成AVCaptureDeviceInput对象
    AVCaptureDeviceInput *videoInput = [AVCaptureDeviceInput deviceInputWithDevice:videoDevice error:error];
    
    //判断videoInput是否有效
    if (videoInput)
    {
        //canAddInput：测试是否能被添加到会话中
        if ([self.captureSession canAddInput:videoInput])
        {
            //将videoInput 添加到 captureSession中
            [self.captureSession addInput:videoInput];
            self.activeVideoInput = videoInput;
        }
    }else
    {
        return NO;
    }
    
    //选择默认音频捕捉设备 即返回一个内置麦克风
    AVCaptureDevice *audioDevice = [AVCaptureDevice defaultDeviceWithMediaType:AVMediaTypeAudio];
    
    //为这个设备创建一个捕捉设备输入
    AVCaptureDeviceInput *audioInput = [AVCaptureDeviceInput deviceInputWithDevice:audioDevice error:error];
   
    //判断audioInput是否有效
    if (audioInput) {
        
        //canAddInput：测试是否能被添加到会话中
        if ([self.captureSession canAddInput:audioInput])
        {
            //将audioInput 添加到 captureSession中
            [self.captureSession addInput:audioInput];
        }
    }else
    {
        return NO;
    }

    //AVCaptureStillImageOutput 实例 从摄像头捕捉静态图片
    self.imageOutput = [[AVCaptureStillImageOutput alloc]init];
    
    //配置字典：希望捕捉到JPEG格式的图片
    self.imageOutput.outputSettings = @{AVVideoCodecKey:AVVideoCodecJPEG};
    
    //输出连接 判断是否可用，可用则添加到输出连接中去
    if ([self.captureSession canAddOutput:self.imageOutput])
    {
        [self.captureSession addOutput:self.imageOutput];
        
    }
    
    
    //创建一个AVCaptureMovieFileOutput 实例，用于将Quick Time 电影录制到文件系统
    self.movieOutput = [[AVCaptureMovieFileOutput alloc]init];
    
    //输出连接 判断是否可用，可用则添加到输出连接中去
    if ([self.captureSession canAddOutput:self.movieOutput])
    {
        [self.captureSession addOutput:self.movieOutput];
    }
    
    
    self.videoQueue = dispatch_queue_create("cc.VideoQueue", NULL);
    
    return YES;
}
```

启动捕捉会话
```
- (void)startSession {

    //检查是否处于运行状态
    if (![self.captureSession isRunning])
    {
        //使用同步调用会损耗一定的时间，则用异步的方式处理
        dispatch_async(self.videoQueue, ^{
            [self.captureSession startRunning];
        });
        
    }
}
```

停止捕捉会话
```
- (void)stopSession {
    
    //检查是否处于运行状态
    if ([self.captureSession isRunning])
    {
        //使用异步方式，停止运行
        dispatch_async(self.videoQueue, ^{
            [self.captureSession stopRunning];
        });
    }
}
```


<!-- ************************************************ -->
## <a id="content2"></a>处理隐私需求

在这个项目会涉及到摄像头、相册、麦克风。需要给出用户提醒，处理隐私需求

注意：iOS7版本只有特定地区有法律规定才会询问用户是否可以访问设备的相机。而从iOS8.0之后，所有的地区和用户都要在应用程序中取得授权才可以访问相机。

常用的隐私设置 plist 修改

<img src="/images/AudioVideo/av2.png" alt="img">





----------
>  行者常至，为者常成！


