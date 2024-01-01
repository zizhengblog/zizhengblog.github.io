---
layout: post
title: "4、AV Foundation之视频捕捉-切换摄像头"
date: 2013-02-04
description: ""
tag: 音视频
---






- [文章摘抄自：https://www.jianshu.com/p/f61732835e6b，用于记录学习](https://www.jianshu.com/p/f61732835e6b)





## 目录

* [简介](#content1)
* [实现](#content2)




<!-- ************************************************ -->
## <a id="content1"></a>简介

 基本上ios设备都具备有前置&后置两个摄像头。接下来开发的功能是让用户在摄像头之间进行切换。当然，这个知识点除了能在我们所做的这个demo上使用。在我们的开发项目中也是运用的很平常的


<!-- ************************************************ -->
## <a id="content2"></a>实现

**一、摄像头支撑的方法**

```
#pragma mark - Device Configuration   配置摄像头支持的方法

- (AVCaptureDevice *)cameraWithPosition:(AVCaptureDevicePosition)position {
    
    //获取可用视频设备
    NSArray *devicess = [AVCaptureDevice devicesWithMediaType:AVMediaTypeVideo];
    
    //遍历可用的视频设备 并返回position 参数值
    for (AVCaptureDevice *device in devicess)
    {
        if (device.position == position) {
            return device;
        }
    }
    return nil;
    
    
}
```


```
- (AVCaptureDevice *)activeCamera {

    //返回当前捕捉会话对应的摄像头的device 属性
    return self.activeVideoInput.device;
}

```

```
//返回当前未激活的摄像头
- (AVCaptureDevice *)inactiveCamera {

    //通过查找当前激活摄像头的反向摄像头获得，如果设备只有1个摄像头，则返回nil
       AVCaptureDevice *device = nil;
      if (self.cameraCount > 1)
      {
          if ([self activeCamera].position == AVCaptureDevicePositionBack) {
               device = [self cameraWithPosition:AVCaptureDevicePositionFront];
         }else
         {
             device = [self cameraWithPosition:AVCaptureDevicePositionBack];
         }
     }

    return device;
    

}
```


```
//判断是否有超过1个摄像头可用
- (BOOL)canSwitchCameras {

    
    return self.cameraCount > 1;
}
```

```
//可用视频捕捉设备的数量
- (NSUInteger)cameraCount {

     return [[AVCaptureDevice devicesWithMediaType:AVMediaTypeVideo] count];
    
}
```


**二、切换摄像头**

```
//切换摄像头
- (BOOL)switchCameras {

    //判断是否有多个摄像头
    if (![self canSwitchCameras])
    {
        return NO;
    }
    
    //获取当前设备的反向设备
    NSError *error;
    AVCaptureDevice *videoDevice = [self inactiveCamera];
    
    
    //将输入设备封装成AVCaptureDeviceInput
    AVCaptureDeviceInput *videoInput = [AVCaptureDeviceInput deviceInputWithDevice:videoDevice error:&error];
    
    //判断videoInput 是否为nil
    if (videoInput)
    {
        //标注原配置变化开始
        [self.captureSession beginConfiguration];
        
        //将捕捉会话中，原本的捕捉输入设备移除
        [self.captureSession removeInput:self.activeVideoInput];
        
        //判断新的设备是否能加入
        if ([self.captureSession canAddInput:videoInput])
        {
            //能加入成功，则将videoInput 作为新的视频捕捉设备
            [self.captureSession addInput:videoInput];
            
            //将获得设备 改为 videoInput
            self.activeVideoInput = videoInput;
        }else
        {
            //如果新设备，无法加入。则将原本的视频捕捉设备重新加入到捕捉会话中
            [self.captureSession addInput:self.activeVideoInput];
        }
        
        //配置完成后， AVCaptureSession commitConfiguration 会分批的将所有变更整合在一起。
        [self.captureSession commitConfiguration];
    }else
    {
        //创建AVCaptureDeviceInput 出现错误，则通知委托来处理该错误
        [self.delegate deviceConfigurationFailedWithError:error];
        return NO;
    }
    
    
    
    return YES;
}
```







----------
>  行者常至，为者常成！


