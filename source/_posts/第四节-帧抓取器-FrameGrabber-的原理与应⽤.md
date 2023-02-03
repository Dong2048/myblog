---
title: 第四节 帧抓取器(FrameGrabber)的原理与应⽤
date: 2022-02-09 15:13:19
tags: 流媒体
category: 死磕javaCV记录
top_img: https://s4.ax1x.com/2022/02/10/HYMP6U.png
cover: https://s4.ax1x.com/2022/02/10/HYMP6U.png
---
#### 前⾔
上⼀章⼤体讲解了javaCV的结构，本章就具体的FrameGrabber实现⽅式展开探索。
#### FrameGrabber(帧抓取器/采集器)介绍
⽤于采集/抓取视频图像和⾳频采样。封装了检索流信息，⾃动猜测视频解码格式，⾳视频解码等具体API，并把解码完的像 素数 据（可配置像素格式）或⾳频数据保存到
Frame中返回等功能。
#### FrameGrabber的结构
FrameGrabber本身是个抽象类，抽象了所有抓取器的通⽤⽅法和⼀些共⽤属性。
> FrameGrabber的⼦类/实现类包含以下⼏个：
FFmpegFrameGrabber、OpenCVFrameGrabber、IPCameraFrameGrabber、VideoInputFrameGrabber、FlyCapture2FrameGrabber、DC1394FrameGrabber、
RealSenseFrameGrabber、OpenKinectFrameGrabber、OpenKinect2FrameGrabber、PS3EyeFrameGrabber
>
#### ⼏种FrameGrabber⼦类实现介绍
FFmpegFrameGrabber：强⼤到离谱的⾳视频处理/计算机视觉/图像处理库，视觉和图像处理这块没有opencv强⼤；  
可以⽀持⽹络摄像机：udp、rtp、rtsp和rtmp等，⽀持本机设备，⽐如屏幕画⾯、摄像头、⾳频话筒设备等等、还⽀持视频⽂件、⽹络流（flv、rtmp、http⽂件流、hls、等等等）    
OpenCVFrameGrabber：Intel开源的opencv计算机视觉/图像处理库，也⽀持读取摄像头、流媒体等，但是⼀般常⽤于读取图⽚，处理图像，流媒体⾳视频这块功能没有ffmpeg强⼤；  
也⽀持rtsp视频流（不⽀持⾳频），本地摄像机设备画⾯采集、视频⽂件、⽹络⽂件、图⽚等等。  
IPCameraFrameGrabber：基于openCV调⽤“⽹络摄像机”。可以直接⽤openCV调⽤，所以⼀般也不需要⽤这个抓取器，例如这个⽹络摄像机地址：
http://IP:port/videostream.cgi?user=admin&pwd=12345&.mjpg；
VideoInputFrameGrabber：只⽀持windows，⽤来读取通过USB连接的摄像头/相机。videoinput库已经被内置到opencv中了，另外videoinput调⽤摄像机的原理是dshow⽅式，
FFmpegFrameGrabber和OpenCVFrameGrabber都⽀持dshow⽅式读取摄像机/相机，所以这个库⼀般⽤不到；  
FlyCapture2FrameGrabber：通过USB3.1,USB2.0、GigE和FireWire（1394）四种⽅式连接的相机，⽐videoinput要慢⼀些；  
DC1394FrameGrabber：另⼀个⽀持MAC的FireWire（1394）外接摄像头/相机，这种接⼝现在已经不常⻅了；  
RealSenseFrameGrabber：⽀持Intel的3D实感摄像头，基于openCV；  
OpenKinectFrameGrabber：⽀持Xbox Kinect v1体感摄像头，通过这个采集器可⽀持mac\linux\windows上使⽤Kinect v1；  
OpenKinect2FrameGrabber：⽤来⽀持Xbox Kinect v2（次世代版）体感摄像头；  
PS3EyeFrameGrabber：没错就是那个sony的PS3，⽤来⽀持PS3的体感摄像头。  
#### FrameGrabber及其实现类的⼀般使⽤流程
> new初始化及初始化设置-->start（⽤于启动⼀些初始化和解析操作）-->循环调⽤grab()获取⾳视频-->stop（销毁所有对象，回收内存）
> 
1、先通过各种new实现类进⾏初始化以及⼀些设置（⽐如设置分辨率、帧率等等）  
2、然后调⽤start()，start中调⽤了⼀系列的解析操作，所以会很耗时，尤其是当读取流媒体时，根据当前⽹络状态情况阻塞时间会⻓⼀些。  
3、调⽤start()⽅法时会等待（阻塞）⼀会⼉就可以获取到源的信息（如果时视频源，这时就可以获取到视频源的相关信息了，⽐如格式，编码，分辨率和码率等等属性）
我们⼀般在这⼀步的时候把⼀些需要⽤到的属性存起来，硫代Recorder或Filter或者进⾏其他图像处理等操作时使⽤。  
4、通过for循环不断的调⽤grab()获取视频帧或者⾳频帧
> FrameGrabber中有⼏种grab：
（1） Frame grab();//通⽤获取解码后的视频图像像素和⾳频采样，⼀般的FrameGrabber⼦类实现只有这个，特殊的⼦类实现会多⼏种获取⽅法。
（2）Frame grabFrame();//FFmpegFrameGrabber特有，等同于上⾯的grab()
（3） Frame grabImage();//FFmpegFrameGrabber特有，只获取解码后的视频图像像素
（4）Frame grabSamples();//FFmpegFrameGrabber特有，只获取⾳频采样
（5）Frame grabKeyFrame();//FFmpegFrameGrabber特有，只获取关键帧
（6）Avpacket grabPacket();//FFmpegFrameGrabber特有，获取解复⽤的⾳/视频帧，也就是grabPacket()可以获取没有经过解码的⾳/视频帧。
（7）void grab()后还需要再调⽤getVideoImage()、getIRImage()和getDepthImage();//这是OpenKinectFrameGrabber特有的来获取体感摄像头图像的⽅法
（8）grabDepth()、grabIR()和grabVideo()//这是OpenKinect2FrameGrabber和RealSenseFrameGrabber特有的调⽤体感摄像头的⽅法
（9）grabBufferedImage()//是IPCameraFrameGrabber⽤于调⽤⽹络摄像机图像的⽅法
（10）grab_RGB4()和grab_raw()//是PS3EyeFrameGrabber⽤于调⽤体感摄像头图像的⽅法
> 
这⾥⽐较特殊的是grab()⽅法，因为它获取到的Frame可能会同时存放⾳频和视频，有时候我们要这⾥⾯的⾳频和视频单独抽取出来进⾏处理，就需要通过两种办法判断Frame的
类型：
>第⼀种判断⽅法（如果视频图像不为空，则显示）：
EnumSet<Type> videoOrAudio=frame.getTypes();
if(hasShow&&videoOrAudio.contains(Type.VIDEO)) {undefined
//可以在这⾥添加⽔印或者进⾏进⾏异步的图像处理等操作，⽐如⼈脸检测识别，⻋牌检测等等操作
canvas.showImage(frame);// 显示画⾯
}
第⼆种判断（如果frame中的image不为空就表示有图像，有则显示）：
if(hasShow&&frame.image!=null&&frame.image.length>1) {undefined
//可以在这⾥添加⽔印或者进⾏进⾏异步的图像处理等操作，⽐如⼈脸检测识别，⻋牌检测等等操作
canvas.showImage(frame);// 显示画⾯
}
recorder.record(frame);//录制或推流操作（frame⾥可以同时包含视频和⾳频）
>
#### FrameGrabber 的使⽤
1、采集/抓取的源：
(1). 读取流媒体⽂件（视频、图⽚、⾳乐和字幕等等⽂件），例如：收流器实现，录制流媒体服务器的rtsp/rtmp视频⽂件(基于javaCV-FFMPEG)  
(2). 拉流（拉取rtsp/rtmp/hls/flv等等流媒体），例如：收流器实现，录制流媒体服务器的rtsp/rtmp视频⽂件(基于javaCV-FFMPEG)  
(3). 采集摄像头图像（⽀持linux/mac/windows/安卓等等多种设备摄像机采集），例如：opencv⽅式调⽤本机摄像头视频；基于dshow调⽤windows摄像头视频和⾳频，想要获  
   取windows屏幕画⾯⾸选gdigrab
(4). 采集设备⾳频采样（⽀持linux/mac/windows/安卓等等多种设备⾳频采集）  
(5). 抓取设备桌⾯屏幕画⾯（⽀持linux/mac/windows/安卓等等多种设备桌⾯屏幕画⾯抓取，⽀持⿏标绘制），例如：基于gdigrab的windows屏幕画⾯抓取/采集（基于javacv的
   屏幕截屏、录屏功能）；基于avfoundation的苹果Mac和ios获取屏幕画⾯及录屏/截屏以及摄像头画⾯和⾳频采样获取实现  
(6). 其他设备/虚拟设备采集（⽐如⽀持Xbox Kinect2\PS3 Eye\RealSense体感摄像头）  
(7). 通过USB等接⼝外接的设备（⽐如USB3.1,USB2.0、GigE和FireWire（1394）等⽅式外接的摄像头等）  
(8). 读取视频裸流，可以通过FFmpegFrameGrabber的构造函数提供构造好的InputStream，可以把裸流通过inputstream传进FrameGrabber中，就可以通过FrameGrabber进⾏
   解复⽤和解码等操作。  
2、⼀些初始化设置
   除了可以设置源的编码格式、像素格式和⾳频格式，⾳频编码等采集属性。  
   还可以设置⽐如视频/桌⾯屏幕分辨率，帧率，⾳频采样率等等属性。  
   示例参考：⾳频重采样以及视频帧像素格式转换  
3、start后就能够读取到源的信息
   ⽐如视频的格式、编码、分辨率、帧率、时⻓和码率等等信息，⾳频可以读取到⾳频的帧率、采样率、⽐特率  
4、抓取/采集到的数据（图像、⾳频）
   调⽤start之后视频⽀持采集到解复⽤（未解码）后的视频帧，视频帧解码后的图像像素数据。  
   ⾳频⽀持解码后的pcm采样等等。  