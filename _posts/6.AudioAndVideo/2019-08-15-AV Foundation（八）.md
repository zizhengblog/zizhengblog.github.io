---
layout: post
title: "8、AV Foundation之视频捕捉-视频捕捉"
date: 2019-08-15
description: ""
tag: 音视频
---






- [文章摘抄自：https://www.jianshu.com/p/09e3d6cb9864，用于记录学习](https://www.jianshu.com/p/09e3d6cb9864)





## 目录

* [前言](#content1)
* [视频捕捉](#content2)







<!-- ************************************************ -->
## <a id="content1"></a>前言

视频内容的捕捉。当设置捕捉会话时，添加一个名为AVCaptureMovieFileOutput的输出。这个了定义了方法将QuickTime 影片捕捉到磁盘。这个类大多数核心功能继承于超类AVCaptureFileOutput.这个超类定义了许多实用功能。比如录制到最长时限或录制到特定文件大小时为止。还可以配置成保留最小可用的磁盘空间。这一点在存储空间有限的移动设备上录制视频时非常重要。

  通常当QuickTime影片准备发布时，影片头的元数据处于文件的开始位置。这样可以让视频播放器快速读取头包含信息，来确定文件的内容、结构和其包含的多个样本的位置。不过，当录制一个QuickTime影片时，直到所有的样片都完成捕捉后才能创建信息头。当录制结束时，创建头数据并将它附在文件结尾处。


<img src="/images/AudioVideo/av3.png" alt="img">


将创建头的过程放在所有影片样本完成捕捉之后存在一个问题，尤其是在移动设备的情况下。如果遇到崩溃或其他中断，比如有电话拨入，则影片头就不会被正确写入，会在磁盘生成一个不可读的影片文件。 AVCaptureMovieFileOutput 提供一个核心功能就是分段捕捉QuickTime影片。


<img src="/images/AudioVideo/av4.png" alt="img">

当录制开始时，在文件最前面写入一个最小化的头信息，随着录制的进行，片段按照一定的周期写入，创建完整的头信息。默认状态下，每10秒写入一个片段，不过这个时间的间隔可以通过修改捕捉设备输出的movieFragentInterval属性来改变。写入片段的方式可以逐步创建完整的QuickTime影片头。这样确保了当遇到应用程序崩溃或中断时，影片仍然会以最好的一个写入片段为终点进行保存。我们用默认的间隔来做这demo，但是如果你可以在你的的APP修改这个值。






<!-- ************************************************ -->
## <a id="content2"></a>视频捕捉

```
#pragma mark - Video Capture Methods 捕捉视频

//判断是否录制状态
- (BOOL)isRecording {

    return self.movieOutput.isRecording;
}

//开始录制
- (void)startRecording {

    if (![self isRecording]) {
        
        //获取当前视频捕捉连接信息，用于捕捉视频数据配置一些核心属性
        AVCaptureConnection * videoConnection = [self.movieOutput connectionWithMediaType:AVMediaTypeVideo];
        
        //判断是否支持设置videoOrientation 属性。
        if([videoConnection isVideoOrientationSupported])
        {
            //支持则修改当前视频的方向
            videoConnection.videoOrientation = [self currentVideoOrientation];
            
        }
        
        //判断是否支持视频稳定 可以显著提高视频的质量。只会在录制视频文件涉及
        if([videoConnection isVideoStabilizationSupported])
        {
            videoConnection.enablesVideoStabilizationWhenAvailable = YES;
        }
        
        
        AVCaptureDevice *device = [self activeCamera];
        
        //摄像头可以进行平滑对焦模式操作。即减慢摄像头镜头对焦速度。当用户移动拍摄时摄像头会尝试快速自动对焦。
        if (device.isSmoothAutoFocusEnabled) {
            NSError *error;
            if ([device lockForConfiguration:&error]) {
                
                device.smoothAutoFocusEnabled = YES;
                [device unlockForConfiguration];
            }else
            {
                [self.delegate deviceConfigurationFailedWithError:error];
            }
        }
        
        //查找写入捕捉视频的唯一文件系统URL.
        self.outputURL = [self uniqueURL];
        
        //在捕捉输出上调用方法 参数1:录制保存路径  参数2:代理
        [self.movieOutput startRecordingToOutputFileURL:self.outputURL recordingDelegate:self];
        
    }
    
    
}

- (CMTime)recordedDuration {
    
    return self.movieOutput.recordedDuration;
}


//写入视频唯一文件系统URL
- (NSURL *)uniqueURL {

    NSFileManager *fileManager = [NSFileManager defaultManager];
    
    //temporaryDirectoryWithTemplateString  可以将文件写入的目的创建一个唯一命名的目录；
    NSString *dirPath = [fileManager temporaryDirectoryWithTemplateString:@"kamera.XXXXXX"];
    
    if (dirPath) {
        
        NSString *filePath = [dirPath stringByAppendingPathComponent:@"kamera_movie.mov"];
        return  [NSURL fileURLWithPath:filePath];
        
    }
    
    return nil;
    
}

//停止录制
- (void)stopRecording {

    //是否正在录制
    if ([self isRecording]) {
        [self.movieOutput stopRecording];
    }
}

#pragma mark - AVCaptureFileOutputRecordingDelegate

- (void)captureOutput:(AVCaptureFileOutput *)captureOutput
didFinishRecordingToOutputFileAtURL:(NSURL *)outputFileURL
      fromConnections:(NSArray *)connections
                error:(NSError *)error {

    //错误
    if (error) {
        [self.delegate mediaCaptureFailedWithError:error];
    }else
    {
        //写入
        [self writeVideoToAssetsLibrary:[self.outputURL copy]];
        
    }
    
    self.outputURL = nil;
    

}

//写入捕捉到的视频
- (void)writeVideoToAssetsLibrary:(NSURL *)videoURL {
    
    //ALAssetsLibrary 实例 提供写入视频的接口
    ALAssetsLibrary *library = [[ALAssetsLibrary alloc]init];
    
    //写资源库写入前，检查视频是否可被写入 （写入前尽量养成判断的习惯）
    if ([library videoAtPathIsCompatibleWithSavedPhotosAlbum:videoURL]) {
        
        //创建block块
        ALAssetsLibraryWriteVideoCompletionBlock completionBlock;
        completionBlock = ^(NSURL *assetURL,NSError *error)
        {
            if (error) {
                
                [self.delegate assetLibraryWriteFailedWithError:error];
            }else
            {
                //用于界面展示视频缩略图
                [self generateThumbnailForVideoAtURL:videoURL];
            }
            
        };
        
        //执行实际写入资源库的动作
        [library writeVideoAtPathToSavedPhotosAlbum:videoURL completionBlock:completionBlock];
    }
}

//获取视频左下角缩略图
- (void)generateThumbnailForVideoAtURL:(NSURL *)videoURL {

    //在videoQueue 上，
    dispatch_async(self.videoQueue, ^{
        
        //建立新的AVAsset & AVAssetImageGenerator
        AVAsset *asset = [AVAsset assetWithURL:videoURL];
        
        AVAssetImageGenerator *imageGenerator = [AVAssetImageGenerator assetImageGeneratorWithAsset:asset];
        
        //设置maximumSize 宽为100，高为0 根据视频的宽高比来计算图片的高度
        imageGenerator.maximumSize = CGSizeMake(100.0f, 0.0f);
        
        //捕捉视频缩略图会考虑视频的变化（如视频的方向变化），如果不设置，缩略图的方向可能出错
        imageGenerator.appliesPreferredTrackTransform = YES;
        
        //获取CGImageRef图片 注意需要自己管理它的创建和释放
        CGImageRef imageRef = [imageGenerator copyCGImageAtTime:kCMTimeZero actualTime:NULL error:nil];
        
        //将图片转化为UIImage
        UIImage *image = [UIImage imageWithCGImage:imageRef];
        
        //释放CGImageRef imageRef 防止内存泄漏
        CGImageRelease(imageRef);
        
        //回到主线程
        dispatch_async(dispatch_get_main_queue(), ^{
            
            //发送通知，传递最新的image
            [self postThumbnailNotifification:image];
            
        });
        
    });
    
}

```



----------
>  行者常至，为者常成！


