---
layout: post
title: "5、AV Foundation之视频捕捉-配置捕捉设备"
date: 2019-08-09
description: ""
tag: 音视频
---


<h6>版权声明：本文为博主原创文章，未经博主允许不得转载。</h6>



- [文章摘抄自：https://www.jianshu.com/p/2b792812f17e，用于记录学习](https://www.jianshu.com/p/2b792812f17e)





## 目录

* [前言](#content1)
* [点击聚焦方法的实现](#content2)
* [点击曝光的方法实现](#content3)
* [重新设置对焦&曝光](#content4)






<!-- ************************************************ -->
## <a id="content1"></a>前言

AVCapture Device 定义了很多方法，让开发者控制ios设备上的摄像头。可以独立调整和锁定摄像头的焦距、曝光、白平衡。对焦和曝光可以基于特定的兴趣点进行设置，使其在应用中实现点击对焦、点击曝光的功能。

  还可以让你控制设备的LED作为拍照的闪光灯或手电筒的使用

  每当修改摄像头设备时，一定要先测试修改动作是否能被设备支持。并不是所有的摄像头都支持所有功能，例如牵制摄像头就不支持对焦操作，因为它和目标距离一般在一臂之长的距离。但大部分后置摄像头是可以支持全尺寸对焦。尝试应用一个不被支持的动作，会导致异常崩溃。所以修改摄像头设备前，需要判断是否支持。比如，将对焦模式设置为自动之前，首先要检查此模式是否被支持。

```
    AVCaptureDevice *device = [self activeCamera];
    
    //是否支持兴趣点对焦 & 是否自动对焦模式
    if (device.isFocusPointOfInterestSupported && [device isFocusModeSupported:AVCaptureFocusModeAutoFocus]) {}
```

当验证这一个配置的修改可以支持时，就可以执行实际的设置配置了。修改捕捉设备的基本技巧包括先锁定设备准备配置，执行所需的修改，最后解锁设备。

Mac、iPhone、iPad上的设备都是系统通用的，所以在进行修改前，AVCaptureDevice要求开发者获得设备上的一个排它锁🔒，不这样做会导致应用程序抛出异常。虽然不要求配置完立即释放排它锁🔒，不过如果不释放则会对其他使用同一个资源的应用程序产生副作用，所以大多数时候我们每当配置完成后就释放这个排它锁🔒。


<!-- ************************************************ -->
## <a id="content2"></a>点击聚焦方法的实现

```
- (BOOL)cameraSupportsTapToFocus {
    
    //询问激活中的摄像头是否支持兴趣点对焦
    return [[self activeCamera]isFocusPointOfInterestSupported];
}

- (void)focusAtPoint:(CGPoint)point {
    
    AVCaptureDevice *device = [self activeCamera];
    
    //是否支持兴趣点对焦 & 是否自动对焦模式
    if (device.isFocusPointOfInterestSupported && [device isFocusModeSupported:AVCaptureFocusModeAutoFocus]) {
        
        NSError *error;
        //锁定设备准备配置，如果获得了锁
        if ([device lockForConfiguration:&error]) {
            
            //将focusPointOfInterest属性设置CGPoint
            device.focusPointOfInterest = point;
            
            //focusMode 设置为AVCaptureFocusModeAutoFocus
            device.focusMode = AVCaptureFocusModeAutoFocus;
            
            //释放该锁定
            [device unlockForConfiguration];
        }else{
            //错误时，则返回给错误处理代理
            [self.delegate deviceConfigurationFailedWithError:error];
        }
        
    }
    
}
```

<!-- ************************************************ -->
## <a id="content3"></a>点击曝光的方法实现

```

- (BOOL)cameraSupportsTapToExpose {
    
    //询问设备是否支持对一个兴趣点进行曝光
    return [[self activeCamera] isExposurePointOfInterestSupported];
}

static const NSString *THCameraAdjustingExposureContext;

- (void)exposeAtPoint:(CGPoint)point {

    
    AVCaptureDevice *device = [self activeCamera];
    
    AVCaptureExposureMode exposureMode =AVCaptureExposureModeContinuousAutoExposure;
    
    //判断是否支持 AVCaptureExposureModeContinuousAutoExposure 模式
    if (device.isExposurePointOfInterestSupported && [device isExposureModeSupported:exposureMode]) {
        
        [device isExposureModeSupported:exposureMode];
        
        NSError *error;
        
        //锁定设备准备配置
        if ([device lockForConfiguration:&error])
        {
            //配置期望值
            device.exposurePointOfInterest = point;
            device.exposureMode = exposureMode;
            
            //判断设备是否支持锁定曝光的模式。
            if ([device isExposureModeSupported:AVCaptureExposureModeLocked]) {
                
                //支持，则使用kvo确定设备的adjustingExposure属性的状态。
                [device addObserver:self forKeyPath:@"adjustingExposure" options:NSKeyValueObservingOptionNew context:&THCameraAdjustingExposureContext];
                
            }
            
            //释放该锁定
            [device unlockForConfiguration];
            
        }else
        {
            [self.delegate deviceConfigurationFailedWithError:error];
        }
        
        
    }
    
}

- (void)observeValueForKeyPath:(NSString *)keyPath
                      ofObject:(id)object
                        change:(NSDictionary *)change
                       context:(void *)context {

    //判断context（上下文）是否为THCameraAdjustingExposureContext
    if (context == &THCameraAdjustingExposureContext) {
        
        //获取device
        AVCaptureDevice *device = (AVCaptureDevice *)object;
        
        //判断设备是否不再调整曝光等级，确认设备的exposureMode是否可以设置为AVCaptureExposureModeLocked
        if(!device.isAdjustingExposure && [device isExposureModeSupported:AVCaptureExposureModeLocked])
        {
            //移除作为adjustingExposure 的self，就不会得到后续变更的通知
            [object removeObserver:self forKeyPath:@"adjustingExposure" context:&THCameraAdjustingExposureContext];
            
            //异步方式调回主队列，
            dispatch_async(dispatch_get_main_queue(), ^{
                NSError *error;
                if ([device lockForConfiguration:&error]) {
                    
                    //修改exposureMode
                    device.exposureMode = AVCaptureExposureModeLocked;
                    
                    //释放该锁定
                    [device unlockForConfiguration];
                    
                }else
                {
                    [self.delegate deviceConfigurationFailedWithError:error];
                }
            });
            
        }
        
    }else
    {
        [super observeValueForKeyPath:keyPath ofObject:object change:change context:context];
    }
    
    
}
```


<!-- ************************************************ -->
## <a id="content4"></a>重新设置对焦&曝光

```
//重新设置对焦&曝光
- (void)resetFocusAndExposureModes {

    
    AVCaptureDevice *device = [self activeCamera];
    
    
    
    AVCaptureFocusMode focusMode = AVCaptureFocusModeContinuousAutoFocus;
    
    //获取对焦兴趣点 和 连续自动对焦模式 是否被支持
    BOOL canResetFocus = [device isFocusPointOfInterestSupported]&& [device isFocusModeSupported:focusMode];
    
    AVCaptureExposureMode exposureMode = AVCaptureExposureModeContinuousAutoExposure;
    
    //确认曝光度可以被重设
    BOOL canResetExposure = [device isFocusPointOfInterestSupported] && [device isExposureModeSupported:exposureMode];
    
    //回顾一下，捕捉设备空间左上角（0，0），右下角（1，1） 中心点则（0.5，0.5）
    CGPoint centPoint = CGPointMake(0.5f, 0.5f);
    
    NSError *error;
    
    //锁定设备，准备配置
    if ([device lockForConfiguration:&error]) {
        
        //焦点可设，则修改
        if (canResetFocus) {
            device.focusMode = focusMode;
            device.focusPointOfInterest = centPoint;
        }
        
        //曝光度可设，则设置为期望的曝光模式
        if (canResetExposure) {
            device.exposureMode = exposureMode;
            device.exposurePointOfInterest = centPoint;
            
        }
        
        //释放锁定
        [device unlockForConfiguration];
        
    }else
    {
        [self.delegate deviceConfigurationFailedWithError:error];
    }
    
    
    
    
}
```



----------
>  行者常至，为者常成！


