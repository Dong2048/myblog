---
title: 第三节 调用opencv原生API和JavaCV是如何封装了opencv的图像处理操作
date: 2022-02-09 14:43:23
tags: 流媒体
category: 死磕javaCV记录
top_img: https://s4.ax1x.com/2022/02/10/HYMClT.jpg
cover: https://s4.ax1x.com/2022/02/10/HYMClT.jpg
---
#### ⼀、前⾔
调⽤FFmpeg原⽣API和JavaCV封装了FFmpeg的⾳视频操作结构如图： 

![avatar](https://s4.ax1x.com/2022/02/09/HGAHAS.png)

#### ⼆、javaCPP直接调⽤opencv 的API
我们通过《实时视频添加⽂字⽔印并截取视频图像保存成图⽚》作为实例来阐述，实例地址：
openCV图像处理之1：实时视频添加⽂字⽔印并截取视频图像保存成图⽚，实现⽂字⽔印的字体、位置、⼤⼩、粗度、翻转、平滑等操作
以及javacv进阶opencv图像检测/识别系列
JavaCV进阶opencv图像检测识别：摄像头画⾯⼈脸检测
JavaCV进阶opencv图像检测识别：ffmpeg视频图像画⾯⼈脸检测
这部分内容主要是如何调⽤opencv的API，本系列作为JavaCV⼊⻔指南不会讲解opencv的具体⽤法，如果想要深⼊学习opencv部分，除了博主的opencv系列之外，还可以参考
opencv的官⽅⽂档及实例。
#### 三、JavaCV是如何封装了opencv的⾳视频及图像处理操作？
与 ffmpeg 相似的是，JavaCV把opencv的操作也抽象成了“读取媒体⽂件或地址，循环抓取图像，停⽌”的流程，其中不同的是，opencv可以直接循环读取设备列表：详细请参
考：
opencv图像处理3：使⽤opencv原⽣⽅法遍历摄像头设备及调⽤（⽅便多摄像头遍历及调⽤，相⽐javacv更快的摄像头读取速度和效率，⽅便读取后的图像处理）
JavaCV把opencv的操作分成了两⼤块：OpenCVFrameGrabber和OpenCVFrameRecorder。其中OpenCVFrameGrabber⽤来读取设备、视频流媒体和图⽚⽂件等，⽽
OpenCVFrameRecorder则⽤来录制⽂件。
##### 1、OpenCVFrameGrabber读取设备、媒体⽂件及流
OpenCVFrameGrabber其实内部封装了opencv的VideoCapture操作，⽀持设备、视频⽂件、图⽚⽂件和流媒体地址（rtsp/rtmp等）。
可以通过 ImageMode设置⾊彩模式，⽀持ImageMode.COLOR（⾊彩图）和ImageMode.GRAY（灰度图）
⽀持的格式请参考：The 'MP4' Registration Authority和Video Codecs by FOURCC - fourcc.org这两个列表。
> 注意：opencv并不⽀持⾳频读取和录制等操作，只⽀持视频⽂件、视频流媒体、图像采集设备的画⾯抓取。
另外需要注意的是，读取⾮动态图⽚，只能读取⼀帧。
> 
通过OpenCVFrameRecorder的grab()抓取到的图像是 Frame ，其实javaCV内部通过OpenCVFrameConverter把opencv的Mat转换为了Frame，也即是说，Frame中可以直接获
取Mat或者也可以通过OpenCVFrameConverter实现Mat和Frame的互转。
##### 2、OpenCVFrameGrabber获取的Frame和Mat之间的关系
OpenCVFrameGrabber既然内部封装的是opencv的VideoCapture，那么获取的数据结构应该是Mat，通过OpenCVFrameGrabber源码也证实了这⼀点，那么我们如何直接获取
Mat呢？
实际上在OpenCVFrameGrabber中把获取到的Mat转换为Frame，并且在Frame中使⽤opaque字段引⽤了Mat结构体。
在上⼀章FFmpegFrameGrabber中也有提到opaque字段的说明，Frame中的opaque字段引⽤了opencv的Mat结构体，⼩伙伴们在使⽤OpenCVFrameGrabber获取Frame的时
候不需要再进⾏Frame和Mat转换，只需要使⽤
> Mat img=(Mat)frame.opaque;
> 
即可获得Mat结构体。
##### 3、OpenCVFrameRecorder录制媒体⽂件
opencv的录制⽀持的视频编码fourCC的编码集请参考：The 'MP4' Registration Authority和Video Codecs by FOURCC - fourcc.org这两个列表。
OpenCVFrameRecorder主要封装了opencv的VideoWriter模块，⽤来实现视频流媒体的录制操作，⽀持的格式同样参考：The 'MP4' Registration Authority和Video Codecs by
FOURCC - fourcc.org这两个列表。
通过循环recordFrame就可以录制视频流媒体，当然如果是进⾏图像处理操作，得到的是mat，就可以通过OpenCVFrameConverter把Mat转换成Frame即可进⾏record()录制操
作。
##### 4、OpenCVFrameConverter进⾏Mat、 IplImage和Frame的互转
由于我们使⽤opencv需要进⾏图像处理等操作，处理完得到的是Mat或者IplImage，读取和录制却是Frame，所以需要使⽤OpenCVFrameConverter提供的转换操作来完成两个对
象间的转换操作。
> 注意：此转换操作实际上常⽤于FFmpeg获取到的视频图像转换为Mat，因为OpencvFrameGrabber本身获取到的Frame包含Mat的引⽤，参考上⾯的
> 
“OpenCVFrameGrabber获取的Frame和Mat之间的关系”
IplImage与Frame互转
>//把frame转换成IplImage
IplImage convertToIplImage(Frame frame)
//把IplImage转换成frame
Frame convert(IplImage img)
> 
Mat与Frame互转
> //frame转换成Mat
Mat convertToMat(Frame frame)
//mat转换成frame
Frame convert(Mat mat)

