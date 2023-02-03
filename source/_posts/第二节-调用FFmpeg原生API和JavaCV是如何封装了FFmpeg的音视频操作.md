---
title: 第二节 调用FFmpeg原生API和JavaCV是如何封装了FFmpeg的音视频操作
date: 2022-02-09 14:32:31
tags: 流媒体
category: 死磕javaCV记录
top_img: https://s4.ax1x.com/2022/02/10/HYM9pV.jpg
cover: https://s4.ax1x.com/2022/02/10/HYM9pV.jpg
---
#### ⼀、什么是JavaCPP
⼤家知道 FFmpeg 是C语⾔中著名的⾳视频库（注意，不是c++。使⽤c++调⽤ffmpeg库的性能损失与Java⽅式调⽤损耗相差并不⼤）。  
JavaCV利⽤JavaCPP在FFmpeg和Java之间构建了桥梁，我们通过这个桥梁可以⽅便的调⽤FFmpeg，当然这并不是没有损失的，性能损失暂且不提，最主
要问题在于调⽤ffmpeg之于jvm是native⽅法，所以通过ffmpeg创建的结构体实例与常量、⽅法等等都是使⽤堆外内存，都需要像C那样⼿动的释放这些资源
（jvm并不会帮你回收这部分），以此来保证不会发⽣内存溢出/泄露等⻛险。  
>Javapp在Java内部提供了对本地C++的⾼效访问，这与⼀些C/C++编译器与汇编语⾔交互的⽅式不同。不需要发明新的语⾔，⽐如SWIG、SIP、C++、
CLI、Cython或Rython。相反，类似于CPpyy为Python所做的努⼒，它利⽤了Java和C++之间的语法和语义相似性。因为使⽤JNI，因此除了RoboVM之
外，它还适⽤于Java SE的所有实现
#### ⼆、javaCPP直接调⽤FFmpeg的API
我们通过《视频拉流解码成YUVJ420P，并保存为jpg图⽚》作为实例来阐述，
这部分内容主要是如何调⽤FFmpeg的API，本系列作为JavaCV⼊⻔不会讲解FFmpeg的具体⽤法，如果想要深⼊学习FFmpeg部分，可以选择通过查看
FFmpeg的API⼿册ffmpeg.org，或者访问雷霄骅的博客详细学习FFmpeg的使⽤。
#### 三、JavaCV是如何封装了FFmpeg的⾳视频操作？
JavaCV通过JavaCPP调⽤了FFmpeg，并且对FFmpeg复杂的操作进⾏了封装，把视频处理分成了两⼤类：“帧抓取器”（FrameGrabber）和“帧录制器”（⼜
叫“帧推流器”，FrameRecorder）以及⽤于存放⾳视频帧的Frame（FrameFilter暂且不提）。  
整体结构如下：  
视频源---->帧抓取器（FrameGabber） ---->抓取视频帧（Frame）---->帧录制器（FrameRecorder）---->推流/录制---->流媒体服务/录像⽂件
#### #1、帧抓取器（FrameGrabber）
封装了FFmpeg的检索流信息，⾃动猜测视频解码格式，⾳视频解码等具体API，并把解码完的像素数据（可配置像素格式）或⾳频数据保存到Frame中返
回。  
帧抓取器详细剖析：帧抓取器(FrameGrabber)的原理与应⽤  
#### #2、帧录制器/推流器（FrameRecorder）
封装了FFmpeg的⾳视频编码操作和封装操作，把传参过来的Frame中的数据取出并进⾏编码、封装、发送等操作流程。  
帧录制器/推流器详细剖析：帧录制器/推流器（FrameRecorder）的原理与应⽤  
#### #3、过滤器（FrameFilter）
FrameFilter的实现类其实只有FFmpegFrameFilter，因为只有ffmpeg⽀持⾳视频的过滤器操作，主要封装了简单的ffmpeg filter操作。  
过滤器详细剖析：帧过滤器(FrameFilter)的原理与应⽤  
#### #4、Frame
⽤于存放解码后的视频图像像素和⾳频采样数据（如果没有配置FrameGrabber的像素格式和⾳频格式，那么默认解码后的视频格式是yuv420j，⾳频则是pcm
采样数据）。  
⾥⾯包含解码后的图像像素数据，⼤⼩（分辨率）、⾳频采样数据，⾳频采样率，⾳频通道（单声道、⽴体声等等）等等数据  
Frame⾥⾯的⼀个字段opaque引⽤AVFrame、AVPacket、Mat等数据，也即是说，如果你是解码后获取的Frame，⾥⾯存放的属性找不到你需要的，可以从
opaque属性中取需要的AVFrame原⽣属性。  
例如：
>Frame frame = grabber.grabImage();//获取视频解码后的图像像素，也就是说这时的Frame中的opaque存放的是AVFrame
AVFrame avframe=(AVFrame)frame.opaque;//把Frame直接强制转换为AVFrame
long lastPts=avframe.pts();
System.err.println("显示时间："+lastPts);
> 
补充：
FFmpeg中两个重要的结构体：AVPacket和AVFrame。  
AVPacket是ffmpeg中存放解复⽤（未解码）的⾳视频帧的结构体，视频只有可能是⼀帧，⼤⼩不定（分为关键帧/I帧、P帧和B帧三种视频帧）。AVPacket
属性除了包含⾳视频帧以外，还包含：pts(显示时间)、dts(解码时间)、duration（持续时⻓）、stream_index（表示⾳视频流的通道，⾳频和视频⼀般是分
开的，通过stream_index来区分是视频帧还是⾳频帧）、flags（AV_PKT_FLAG_KEY（值是1）：关键帧，2-损坏数据，4-丢弃数据）、pos（在流媒体中的
位置）、size  
某些情况下，使⽤AVPacket直接推流（不经过转码）的过程称之为：转封装。
AVFrame是ffmpeg中存放解码后的⾳视频图像像素或⾳频采样数据的结构体，⼤部分属性是与Frame相同的，多了像素格式、pts、dts和⾳频布局等等属性。
AVPakcet和AVFrame的使⽤流程如下图所示：  

![avatar](https://s4.ax1x.com/2022/02/09/HGA1O0.png)