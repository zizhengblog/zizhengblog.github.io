---
layout: post
title: "9、AV Foundation之视频捕捉-二维码识别"
date: 2019-08-17
description: ""
tag: 音视频
---






- [文章摘抄自：https://www.jianshu.com/nb/13168678，用于记录学习](https://www.jianshu.com/nb/13168678)





## 目录

* [前言](#content1)
* [代码实现](#content2)




<!-- ************************************************ -->
## <a id="content1"></a>前言

**一、简介**

AV Foundation 魅力远不止做一个系统相机或做一个简单播放器。那我们今天在AV Foundation的媒体捕捉的基础上，来尝试做一次延伸。

AV Foundation 支持对动态识别，不仅可以做人脸识别，也可以识别机器可读代码。AV Foundation 在识别这一块有硬件加速器。所以可以同时扫描10张人脸或机器码。

iOS原生识别二维码：CIDetector ,(Core Image 也可以识别二维码)

iOS原生生成二维码：CIFilter(生成二维码)

第三方：zxing、zbar、LBXSan 


**二、机器可读代码识别**

一维码：UPC-E  、EAN - 8、EAN - 13、Code 39、Code 128、交错式2of5码、ITF(iOS8.0)

二维码：

1.QR 码， 用于移动营销

2.Aztec码，广泛用于航天领域内登机牌

3.PDF417,商品运输应用程序

4.Data Matrix（iOS 8.0):多用于营销


<!-- ************************************************ -->
## <a id="content2"></a>代码实现


**一、二维码图层**

```

#import "THPreviewView.h"

@interface THPreviewView ()<THCodeDetectionDelegate>

@property(strong,nonatomic)NSMutableDictionary *codeLayers;//二维码图层

@end

@implementation THPreviewView

+ (Class)layerClass {

    //目的让视图的备用层成为AVCaptureVideoPreviewLayer 实例
    return [AVCaptureVideoPreviewLayer class];//获的预览图层
}

- (id)initWithFrame:(CGRect)frame {
    self = [super initWithFrame:frame];
    if (self) {
        [self setupView];
    }
    return self;
}

- (id)initWithCoder:(NSCoder *)coder {
    self = [super initWithCoder:coder];
    if (self) {
        [self setupView];
    }
    return self;
}

- (void)setupView {

    //保存一组表示识别编码的几何信息图层。
    _codeLayers = [NSMutableDictionary dictionary];
    
    //设置图层的videoGravity 属性，保证宽高比在边界范围之内
    self.previewLayer.videoGravity = AVLayerVideoGravityResizeAspect;
    
}

- (AVCaptureSession*)session {

    return [[self previewLayer]session];
}

//重写setSession 方法，将AVCaptureSession 作为预览层的session 属性
- (void)setSession:(AVCaptureSession *)session {
    
    self.previewLayer.session = session;
}

- (AVCaptureVideoPreviewLayer *)previewLayer {

    return (AVCaptureVideoPreviewLayer *)self.layer;
}

//元数据转换
- (void)didDetectCodes:(NSArray *)codes {
    
    //保存转换完成的元数据对象
    NSArray *transformedCodes = [self transformedCodesFromCodes:codes];
    
    //从codeLayers字典中获得key，用来判断那个图层应该在方法尾部移除
    NSMutableArray *lostCodes = [self.codeLayers.allKeys mutableCopy];
    
    //遍历数组
    for (AVMetadataMachineReadableCodeObject *code in transformedCodes) {
        
        //获得code.stringValue
        NSString *stringValue = code.stringValue;
        
        if (stringValue) {
         
            [lostCodes removeObject:stringValue];
            
        }else
        {
            continue;
        }
        
        //根据当前的stringValue 查找图层。
        NSArray *layers = self.codeLayers[stringValue];
        
        //如果没有对应的类目
        if (!layers) {
            
            //新建图层 方、圆
            layers = @[[self makeBoundsLayer],[self makeCornersLayer]];
            
            //将图层以stringValue 为key 存入字典中
            self.codeLayers[stringValue] = layers;
            
            //在预览图层上添加 图层0、图层1
            [self.previewLayer addSublayer:layers[0]];
            [self.previewLayer addSublayer:layers[1]];
        }
        
        //创建一个和对象的bounds关联的 UIBezierPath
        //画方框
        CAShapeLayer *boundsLayer = layers[0];
        boundsLayer.path = [self bezierPathForBounds:code.bounds].CGPath;
        
        //对于cornersLayer 构建一个CGPath
        CAShapeLayer *cornersLayer = layers[1];
        cornersLayer.path = [self bezierPathForCorners:code.corners].CGPath;
    }
    
    //遍历lostCodes
    for (NSString *stringValue in lostCodes) {
        
        //将里面的条目图层从 previewLayer 中移除
        for (CALayer *layer in self.codeLayers[stringValue]) {
         
            [layer removeFromSuperlayer];
            
        }
        //数组条目中也要移除
        [self.codeLayers removeObjectForKey:stringValue];
    }
    
}

- (NSArray *)transformedCodesFromCodes:(NSArray *)codes {

    
    NSMutableArray *transformedCodes = [NSMutableArray array];
    
    //遍历数组
    for (AVMetadataObject *code in codes) {
        
        //讲设备坐标空间元数据对象 转换为 视图坐标空间对象
        AVMetadataObject *transformedCode = [self.previewLayer transformedMetadataObjectForMetadataObject:code];
        
        //将转换好的数据 添加到 数组中
        [transformedCodes addObject:transformedCode];
        
    }
    
    //返回已经处理好的数据
    return transformedCodes;
}

- (UIBezierPath *)bezierPathForBounds:(CGRect)bounds {

    //绘制一个方框
    return [UIBezierPath bezierPathWithOvalInRect:bounds];
    
}

//CAShapeLayer 是CALayer 子类，用于绘制UIBezierPath  绘制boudns矩形
- (CAShapeLayer *)makeBoundsLayer {

    CAShapeLayer *shapeLayer = [CAShapeLayer layer];
    shapeLayer.strokeColor = [[UIColor colorWithRed:0.95f green:0.75f blue:0.06f alpha:1.0f]CGColor];
    shapeLayer.fillColor = nil;
    shapeLayer.lineWidth = 4.0f;
    return shapeLayer;

}

//CAShapeLayer 是CALayer 子类，用于绘制UIBezierPath  绘制coners路径
- (CAShapeLayer *)makeCornersLayer {

    CAShapeLayer *cornerLayer = [CAShapeLayer layer];
    cornerLayer.lineWidth = 2.0f;
    cornerLayer.strokeColor  = [UIColor colorWithRed:0.172f green:0.671f blue:0.48f alpha:1.000f].CGColor;
    cornerLayer.fillColor = [UIColor colorWithRed:0.190f green:0.753f blue:0.489f alpha:0.5f].CGColor;
    
    return cornerLayer;
}

- (UIBezierPath *)bezierPathForCorners:(NSArray *)corners {
    //创建一个空的UIBezierPath
    UIBezierPath *path = [UIBezierPath bezierPath];
    
    //遍历数组中的条目，为每个条目构建一个CGPoint
     for (int i = 0; i < corners.count; i++)
    {
        CGPoint point = [self pointForCorner:corners[i]];
        
        if (i == 0) {
            [path moveToPoint:point];
        }else
        {
            [path addLineToPoint:point];
        }
    }

    [path closePath];
    return  path;

}

//从字典corner 中拿到需要的点point
- (CGPoint)pointForCorner:(NSDictionary *)corner {

    CGPoint point;
    CGPointMakeWithDictionaryRepresentation((CFDictionaryRef)corner, &point);
    return point;
    
    
}

@end
```

**二、二维码数据获取**

```

#import "THCameraController.h"
#import <AVFoundation/AVFoundation.h>

@interface THCameraController ()<AVCaptureMetadataOutputObjectsDelegate>

@property(strong,nonatomic)AVCaptureMetadataOutput *metadataOutput; //通过代理方法，拿到接收元数据时的通知

@end


@implementation THCameraController

- (NSString *)sessionPreset {
    
    //重写sessionPreset方法，可以选择最适合应用程序捕捉预设类型。
    //苹果公司建议开发者使用最低合理解决方案以提高性能
    return AVCaptureSessionPreset640x480;
}

- (BOOL)setupSessionInputs:(NSError *__autoreleasing *)error {

    //设置相机自动对焦，这样可以在任何距离都可以进行扫描。
    BOOL success = [super setupSessionInputs:error];
    if(success)
    {
        //判断是否能自动聚焦
        if (self.activeCamera.autoFocusRangeRestrictionSupported) {
            
            //锁定设备
            if ([self.activeCamera lockForConfiguration:error]) {
                
                //自动聚焦
                /*
                    iOS 7.0新增属性 允许使用范围约束来对功能进行定制。
                  因为扫描条码，距离都比较近。所以AVCaptureAutoFocusRangeRestrictionNear，
                 通过缩小距离，来提高识别成功率。
                 */
                self.activeCamera.autoFocusRangeRestriction = AVCaptureAutoFocusRangeRestrictionNear;
                
                //释放排他锁
                [self.activeCamera  unlockForConfiguration];
            }
        }
    }

    return YES;
}

- (BOOL)setupSessionOutputs:(NSError **)error {

    //获取输出设备
    self.metadataOutput = [[AVCaptureMetadataOutput alloc]init];
    
    //判断是否能添加输出设备
    if ([self.captureSession canAddOutput:self.metadataOutput]) {
        
        //添加输出设备
        [self.captureSession addOutput:self.metadataOutput];
        
        dispatch_queue_t mainQueue = dispatch_get_main_queue();
        
        //设置委托代理
        [self.metadataOutput setMetadataObjectsDelegate:self queue:mainQueue];
        
        //指定扫描对是OR码 & Aztec 码 (移动营销)
        NSArray *types = @[AVMetadataObjectTypeQRCode,AVMetadataObjectTypeAztecCode,AVMetadataObjectTypeDataMatrixCode,AVMetadataObjectTypePDF417Code];
        
        self.metadataOutput.metadataObjectTypes = types;
        
    }else
    {
        //错误时，存储错误信息
        NSDictionary *userInfo = @{NSLocalizedDescriptionKey:@"Faild to add metadata output."};
        *error = [NSError errorWithDomain:THCameraErrorDomain code:THCameraErrorFailedToAddOutput userInfo:userInfo];
    
        return NO;
        
        
    
    }
    
    
    
    return YES;
}


//委托代理回掉。处理条码
- (void)captureOutput:(AVCaptureOutput *)captureOutput
didOutputMetadataObjects:(NSArray *)metadataObjects
       fromConnection:(AVCaptureConnection *)connection {
    
    if (metadataObjects.count > 0) {
        
        NSLog(@"%@",metadataObjects[0]);
        
        /*
         <AVMetadataMachineReadableCodeObject: 0x17002db20, type="org.iso.QRCode", bounds={ 0.4,0.4 0.1x0.2 }>corners { 0.4,0.6 0.6,0.6 0.6,0.4 0.4,0.4 }, time 122373330766250, stringValue ""http://www.echargenet.com/portal/csService/html/app.html
         */

    }
    
    
    //获取了
    [self.codeDetectionDelegate didDetectCodes:metadataObjects];

}



@end

```




----------
>  行者常至，为者常成！


