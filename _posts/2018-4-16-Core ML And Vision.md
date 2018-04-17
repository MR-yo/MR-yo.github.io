---
layout: post
author: syea
title: Core ML And Vision 
# subtitle:
# header-img: 
# header-mask:  
# catalog: 
# multilingual: 
tags:
    - iOS
    - Objective-C
---

# Core ML And Vision 


### Core ML

Core ML 框架只是将机器学习训练好的模型拿来应用，本身并不涉及机器学习的运行环境和过程。<br>
Core ML 可以看做一个模型的转换器，可以将一个 mlmodel 格式的模型文件自动生成一些类和方法，可以直接使用这些类去做分析，让我们更简单的在app中使用训练好的模型。<br>

主要实现步骤:<br>
1. 获得机器学习的 model
2. 将 model 拖入工程，调用 API

关于 Model，在官网有一些现成的，能力强的也可以自己写，官方配了转换工具。
举个图像识别的🌰:
```
- (void)startCalculateByCoreML  
{
    // 拿到目标图片
    UIImage *targetImage = xxx;

    // 图片转换为 model 输入所需要的格式，不同的 model 输入数据的配置也不同
    UIImage *scaledImage = [targetImage scaleToSize:CGSizeMake(224, 224)];
    CVPixelBufferRef buffer = [targetImage pixelBufferFromCGImage:scaledImage];

    // 实例化 model
    MobileNet *model = [[MobileNet alloc] init];

    // 创建输入数据
    MobileNetInput *input = [[MobileNetInput alloc] initWithImage:buffer];

    // 得到输出数据,不同的 model 输出数据格式也不同
    NSError *error;
    MobileNetOutput *output = [model predictionFromFeatures:input error:&error];

    // 展示结果
    NSNumber *prob = [output.classLabelProbs valueForKey:output.classLabel];
    ...
    ...
}
```

![http://owlvwomsh.bkt.clouddn.com/object.gif](http://owlvwomsh.bkt.clouddn.com/object.gif)

### Vision

Vision 是伴随 iOS 11 推出的基于Core ML的图形处理框架。运用高性能图形处理和视觉技术，可以对图像和视频进行人脸检测、特征点检测和场景识别等，相当于内置了一些Model。

1. Vision 与 CoreModel 一起使用
```
- (void)startCalculateByVisionAndCoreML
{
    // 获取目标图片
    UIImage *targetUIImage = self.selectImage.image;
    CIImage *targetCIImage = [[CIImage alloc] initWithImage:targetUIImage];
    
    // 实例化机器学习的 model
    MobileNet *probModel = [[MobileNet alloc] init];

    // 通过 Vision API 转换为 Vision 的 Model，此处 Vision 知道了该 Model 需要输入什么，输出什么。
    NSError *error;
    VNCoreMLModel *vnmlModel = [VNCoreMLModel modelForMLModel:probModel.model error:&error];
 
    // 实例化 VNCoreMLRequest
    VNCoreMLRequest *vnmlRequest = [[VNCoreMLRequest alloc] initWithModel:vnmlModel completionHandler:^(VNRequest * _Nonnull request, NSError * _Nullable error) {

        // 获取识别结果
        VNClassificationObservation *maxObservation = nil;
        for (VNClassificationObservation *observationResult in request.results) {
            maxObservation = maxObservation.confidence > observationResult.confidence ? maxObservation : observationResult;
        }
        ...
        ...

    }];

    // 实例化 VNImageRequestHandler，传入需要识别的目标图片，这个时候就不需要适配 model 的输入参数了，Vision 会帮你处理。
    VNImageRequestHandler *handler = [[VNImageRequestHandler alloc] initWithCIImage:targetCIImage options:@{}];

    // 发送识别请求
    [handler performRequests:@[vnmlRequest] error:&error];
}
```
2. 仅使用 Vision
```
- (void)startCalculateByVision
{
    // 获取目标图片
    UIImage *targetUIImage = self.selectIamge.image;
    CIImage *targetCIImage = [[CIImage alloc] initWithImage:targetUIImage];
    
    // 实例化 VNImageRequestHandler，并传入需要识别的图片
    VNImageRequestHandler *handler = [[VNImageRequestHandler alloc] initWithCIImage:targetCIImage options:@{}];
    
    /** 
     * 实例化 VNImageBasedRequest
     * **VNDetectBarcodesRequest**  识别条形码，返回 VNBarcodeObservation
     * **VNDetectFaceLandmarksRequest**   人脸特征识别，返回 VNFaceObservation
     * **VNDetectFaceRectanglesRequest**  人脸识别，返回 VNFaceObservation
     * **VNDetectHorizonRequest**   地平线角度检测？，返回 VNHorizonObservation
     * **VNDetectRectanglesRequest**  物体检测和追踪，返回 VNRectangleObservation
     * **VNDetectTextRectanglesRequest**  文本检测，返回 VNTextObservation
     */
    VNImageBasedRequest *vnImageBaseRequest = [[VNDetectFaceLandmarksRequest alloc] initWithCompletionHandler:^(VNRequest * _Nonnull request, NSError * _Nullable error) {
 
        //获取识别结果
        ...
        ...

    }];
    
    // 发送识别请求
    NSError *error;
    [handler performRequests:@[vnImageBaseRequest] error:&error];
}
```
![http://owlvwomsh.bkt.clouddn.com/face.gif](http://owlvwomsh.bkt.clouddn.com/face.gif)

### 总结

机器学习的精确性最根本还是在于训练出的模型，Core ML 只是一个使用的工具。Vision 内置了一些 model，封装了一些 API，使得我们可以在客户端做一些图像识别、物体追踪的操作。就人脸识别来说，利用 Vision 我们可以拿到人脸的65的特征点，但是要得出这个人脸是谁的，就需要一些识别算法来实现，还需要一个存有相关人脸信息的数据库。

优势:
1. 只要有优秀的训练好的 model，就可以利用 Core ML 或者 Vision 在客户端实现快速的机器学习应用。相较于客户端获取资源，服务器解析并返回的方式，更加高效。
2. 高度封装的 API，就客户端而言学习成本低。

劣势:
1. 版本需要在 iOS 11 以上。
2. 需要有训练好的model（门槛比较高），否则只有内置的相关功能。



