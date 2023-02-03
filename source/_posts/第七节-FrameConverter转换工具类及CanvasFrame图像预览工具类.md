---
title: 第七节 FrameConverter转换工具类及CanvasFrame图像预览工具类
date: 2022-02-09 15:45:57
tags: 流媒体
category: 死磕javaCV记录
top_img: https://s4.ax1x.com/2022/02/10/HYQ3vT.jpg
cover: https://s4.ax1x.com/2022/02/10/HYQ3vT.jpg
---
#### 前⾔
再此章之前，我们已经详细介绍和剖析了javacv的结构和 ffmpeg 和opencv的封装调⽤⽅式，以及javacv中重要的FrameGrabber和FrameRecorder的原理和⽤法，本章是javacv
⼊⻔指南的最后⼀章，主要介绍转换⼯具和图像预览⼯具类。
#### FrameConverter介绍
FrameConverter封装了常⽤的转换操作，⽐如opencv与 Frame 的互转、java图像与Frame的互转以及安卓平台的Bitmap图像与Frame互转操作。
#### FrameConverter的⼦类
> AndroidFrameConverter、Java2DFrameConverter、JavaFXFrameConverter、LeptonicaFrameConverter、OpenCVFrameConverter
由于JavaCV的Frame完全是仿照ffmpeg的AVFrame格式设计的，所有AVFrame和Frame不存在互转，它们的数据格式基本是互通的，直接赋值即可。
#### AndroidFrameConverter互转操作
专⻔⽤于安卓平台的转换操作，⽤于将Bitmap和Frame进⾏互转，以及提供了额外的yuv转bgr操作。
> //Frame转换为Bitmap  
Bitmap convert(Frame frame)  
//bitmap转换为frame  
Frame convert(Bitmap bitmap)  
//yuv4:2:0像素转换为BGR像素  
/** Convert YUV 4:2:0 SP (NV21) data to BGR, as received, for example,  
>via {@link Camera.PreviewCallback#onPreviewFrame(byte[],Camera)}.  
  */  
  public Frame convert(byte[] data, int width, int height)  
  #### Java2DFrameConverter互转操作
  提供了Frame和java图像BufferedImage的互转操作。
  > //Frame转BufferedImage图像  
  public BufferedImage getBufferedImage(Frame frame)  
  BufferedImage getBufferedImage(Frame frame, double gamma)//伽⻢值，⽤来调节图像的灰度曲线，与显示设备有关  
  BufferedImage getBufferedImage(Frame frame, double gamma, boolean flipChannels, ColorSpace cs)  
  //BufferedImage图像转Frame  
  Frame getFrame(BufferedImage image)  
  Frame getFrame(BufferedImage image, double gamma)  
  Frame getFrame(BufferedImage image, double gamma, boolean flipChannels)  
  #### JavaFXFrameConverter互转操作
  提供了JavaFX的图像Image和Frame的转换操作。
  > //把javaFX的图像Image转换为javacv的Frame  
  Frame convert(Image f)  
  //把Frame转换为javaFX的Image图像对象  
  Image convert(Frame frame)  
  #### LeptonicaFrameConverter互转操作
  ⽤于Leptonica和tesserac的PIX和Frame的互转，Leptonica是图像识别库google tesserac ocr的依赖库，也即是说该⼯具类⼀般是⽤于tesserac的图像PIX对象与Frame互转操作。  
  > //Frame转tesserac的PIX图像  
  PIX convert(Frame frame)  
  //tesserac的PIX图像转Frame  
  Frame convert(PIX pix)  
  #### OpenCVFrameConverter互转操作
  主要⽤于opencv的IplImage/Mat和Frame的互转操作。  
  > IplImage与Frame互转  
  //把frame转换成IplImage  
  IplImage convertToIplImage(Frame frame)  
  //把IplImage转换成frame  
  Frame convert(IplImage img)  
  Mat与Frame互转  
  //frame转换成Mat  
  Mat convertToMat(Frame frame)  
  //mat转换成frame  
  Frame convert(Mat mat)  
  #### CanvasFrame介绍
  CanvasFrame是⽤于预览Frame图像的⼯具类，但是这个⼯具类的gama值通常是有问题的，所以显示的图像可能会偏⾊，但是不影响最终图像的⾊彩。  
  #### CanvasFrame的原理
  CanvasFrame内部是使⽤的swing的 Canvas 画板操作，使⽤canvas画板绘制图像。  
  #### CanvasFrame的使⽤
  > CanvasFrame canvas = new CanvasFrame("转换apng中屏幕预览");// 新建⼀个窗⼝  
  canvas.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);  
  canvas.setAlwaysOnTop(true)；  
  //显示画⾯，这个操作⽤来显示Frame，⼀般Frame从各个FrameGrabber中获取或者从各个converter转换类中⽽来。  
  canvas.showImage(frame);  
  > 
  死磕javaCV⾄此结束。