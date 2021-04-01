---
layout: post
title: "6、AV Foundation之视频捕捉-调整闪光灯&手电筒模式"
date: 2019-08-11
description: ""
tag: 音视频
---






- [文章摘抄自：https://www.jianshu.com/p/2b792812f17e，用于记录学习](https://www.jianshu.com/p/2b792812f17e)





## 目录

* [前言](#content1)
* [方法实现](#content2)






<!-- ************************************************ -->
## <a id="content1"></a>前言

AVCaptureDevice 类可以让开发者修改摄像头的闪关灯&手电筒模式。设备后面的LED灯当拍摄静态图片时作为闪光灯，而当拍摄视频时用作连续灯光（手电筒）。捕捉设备的flashMode和 touchMode 属性可以被设置为以下3个值中的一个：

AVCapture(Torch/Flash)ModeOn: 闪关灯/手电筒总是开启

AVCapture(Torch/Flash)ModeOff:闪关灯/手电筒总是关闭

AVCapture(Torch/Flash)ModeAuto:系统会基于周围关照环境情况自动关闭或打开LED


<!-- ************************************************ -->
## <a id="content1"></a>方法实现

```
//判断是否有闪光灯
- (BOOL)cameraHasFlash {

    return [[self activeCamera]hasFlash];

}

//闪光灯模式
- (AVCaptureFlashMode)flashMode {

    
    return [[self activeCamera]flashMode];
}

//设置闪光灯
- (void)setFlashMode:(AVCaptureFlashMode)flashMode {

    //获取会话
    AVCaptureDevice *device = [self activeCamera];
    
    //判断是否支持闪光灯模式
    if ([device isFlashModeSupported:flashMode]) {
    
        //如果支持，则锁定设备
        NSError *error;
        if ([device lockForConfiguration:&error]) {

            //修改闪光灯模式
            device.flashMode = flashMode;
            //修改完成，解锁释放设备
            [device unlockForConfiguration];
            
        }else
        {
            [self.delegate deviceConfigurationFailedWithError:error];
        }
        
    }

}

//是否支持手电筒
- (BOOL)cameraHasTorch {

    return [[self activeCamera]hasTorch];
}

//手电筒模式
- (AVCaptureTorchMode)torchMode {

    return [[self activeCamera]torchMode];
}


//设置是否打开手电筒
- (void)setTorchMode:(AVCaptureTorchMode)torchMode {

    
    AVCaptureDevice *device = [self activeCamera];
    
    if ([device isTorchModeSupported:torchMode]) {
        
        NSError *error;
        if ([device lockForConfiguration:&error]) {
            
            device.torchMode = torchMode;
            [device unlockForConfiguration];
        }else
        {
            [self.delegate deviceConfigurationFailedWithError:error];
        }

    }
    
}
```

----------
>  行者常至，为者常成！


