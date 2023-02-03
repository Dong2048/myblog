---
title: 第五节 帧录制器/推流器(FrameRecorder)的原理与应用
date: 2022-02-09 15:21:07
tags: 流媒体
category: 死磕javaCV记录
top_img: https://s4.ax1x.com/2022/02/10/HYKzYq.jpg
cover: https://s4.ax1x.com/2022/02/10/HYKzYq.jpg
---
#### 前⾔
上⼀章⼤体讲解了FrameGrabber（抓取器/采集器），本章就FrameRecorder展开探索。
#### FrameRecorder(录制器/推流器)介绍
⽤于⾳视频/图⽚的封装、编码、推流和录制保存等操作。把从FrameGrabber或者FrameFilter获取的 Frame 中的数据取出并进⾏编码、封装、推流发送等操作流程。为了⽅便
理解和阅读，下⽂开始我们统⼀把FrameRecorder简称为：录制器。
#### FrameRecorder的结构和分析
FrameRecorder与FrameGrabber类似，也是个抽象类，抽象了所有录制器的通⽤⽅法和⼀些共⽤属性。
>FrameRecorder只有两个⼦类实现：FFmpegFrameRecorder和OpenCVFrameRecorder
#### 两个FrameRecorder实现类的介绍
FFmpegFrameRecorder：使⽤ ffmpeg 作为⾳视频编码/图像、封装、推流、录制库。除了⽀持⾳视频录制推流之外，还可以⽀持gif/apng动态图录制，对于⾳视频这块的功能
ffmpeg依然还是⼗分强⼤的保障。
OpenCVFrameRecorder：使⽤opencv的videowriter录制视频，不⽀持像素格式转换，根据opencv的官⽅介绍，如果在ffmpeg可⽤的情况下，opencv的视频功能可能也是基于
ffmpeg的，macos下基于avfunctation。
>总得来说，⾳视频这块还是⾸选ffmpeg的录制器。但是凡事都有例外，由于javacv的包实在太⼤，开发的时候下载依赖都要半天。为了减少程序的体积⼤⼩，如果只需要进
⾏图像处理/图像那个识别的话只使⽤opencv的情况就能解决问题，那就opencvFrameGrabber和OpenCVFrameRecorder配套使⽤吧，毕竟可以省很多空间。
同样的，如果只是做⾳视频流媒体，那么能使⽤ffmpeg解决就不要⽤其他库了，能节省⼀点空间是⼀点，确实javacv整个完整包太⼤了，快要1G了。。。这些都是题外话
了，还是回归正题吧。
> 
FrameRecorder的结构和流程
FrameRecorder的整个代码结构和运作流程很简单
>初始化和设置参数--->start()--->循环record(Frame frame)--->close()
> 
close()包含stop()和release()两个操作，这个要注意。
由于只有两个实现类，就直接分开单独分析了
#### FFmpegFrameRecorder结构和分析
FFmpegFrameRecorder是⽐较复杂的，我们主要它来实现像素格式转换、视频编码和录制推流。
我们把流程分为转封装流程和转码流程
转封装流程：
>FFmpegFrameRecorder初始化-->start()-->循环recordPacket(AVPacket)-->close()
> 
这⾥的AVPacket是未解码的解复⽤视频帧。转封装的示例：转封装在rtsp转rtmp流中的应⽤
转码编码流程：
>FFmpegFrameRecorder初始化-->start()-->循环record(Frame)/recordImage()/recordSamples()-->close()
> 
转码的示例：转流器实现，视频⽂件转gif动态图⽚实现，视频转apng动态图⽚实现
#### FFmpegFrameRecorder初始化及参数说明
FFmpegFrameRecorder初始化⽀持⽂件地址、流媒体推流地址、OutputStream流的形式和imageWidth(视频宽度)、imageHeight（图像⾼度）、audioChannels（⾳频通道，1-
单声道，2-⽴体声）
FFmpegFrameRecorder参数较多，不⼀⼀列举，直接看代码中的参数说明：
>FFmpegFrameRecorder recorder=new FFmpegFrameRecorder(output, width,height,0);
recorder.setVideoCodecName(videoCodecName);//优先级⾼于videoCodec
recorder.setVideoCodec(videoCodec);//只有在videoCodecName没有设置或者没有找到videoCodecName的情况下才会使⽤videoCodec
// recorder.setFormat(format);//只⽀持flv，mp4，3gp和avi四种格式，
flv:AV_CODEC_ID_FLV1;mp4:AV_CODEC_ID_MPEG4;3gp:AV_CODEC_ID_H263;avi:AV_CODEC_ID_HUFFYUV;
recorder.setPixelFormat(pixelFormat);// 只有pixelFormat，width，height三个参数任意⼀个不为空才会进⾏像素格式转换
recorder.setImageScalingFlags(imageScalingFlags);//缩放，像素格式转换时使⽤，但是并不是像素格式转换的判断参数之⼀
recorder.setGopSize(gopSize);//gop间隔
recorder.setFrameRate(frameRate);//帧率
recorder.setVideoBitrate(videoBitrate);
recorder.setVideoQuality(videoQuality);
//下⾯这三个参数任意⼀个会触发⾳频编码
recorder.setSampleFormat(sampleFormat);//⾳频采样格式,使⽤avutil中的像素格式常量，例如：avutil.AV_SAMPLE_FMT_NONE
recorder.setAudioChannels(audioChannels);
recorder.setSampleRate(sampleRate);
recorder.start();
>
#### FFmpegFrameRecorder的start
start中其实做了很多事情：⼀堆初始化操作、打开⽹络流、查找编码器、把format字符转换成对应的videoCodec和videoFormat等等。
#### FFmpegFrameRecorder中的各种record
record(Frame frame)：编码⽅式录制，把已经解码的图像像素编码成对应的编码和格式推流出去  
record(Frame frame, int pixelFormat)：多出⼀个pixelFormat像素格式参数就是因为它会触发像素格式转换，如果像素格式与编码器要求的像素格式不同的时候，就会进⾏转换
像素格式，在转换的时候同时也会转换width、height、imageScalingFlags。  
像素格式转换参考示例：javaCV开发详解之15：视频帧像素格式转换  
recordImage(int width, int height, int depth, int channels, int stride, int pixelFormat, Buffer ... image)：其实上⾯两个都是调⽤的这个⽅法。Buffer[] image是从Frame frame⾥的
image参数获取的，所以很多⼩伙伴问我FrameGrabber解码出来的图像像素在哪⾥？看，它在frame.image⾥。  
recordSamples(Buffer ... samples)：⽤来编码录制⾳频的，它调⽤了下⾯的⽅法，Buffer ... samples是从Frame的frame.samples来，所以如上所述，很多同学来问我
FrameGrabber解码出来的⾳频采样存哪⾥去了？看，它还在Frame⾥⾯。  
recordSamples(int sampleRate, int audioChannels, Buffer ... samples)：⽤来编码录制⾳频的，它会触发重采样，当samples_channels（⾳频通道）、samples_format（⾳频采
样格式）和samples_rate（采样率）任意参数不为空的时候就会触发重采样。  
⾳频重采样示例：javaCV开发详解之14：⾳频重采样  
recordPacket(AVPacket pkt):转封装操作，⽤来放未经过解码的解复⽤视频帧。  
#### FFmpegFrameRecorder中的stop和close
⾮常需要的注意的是，当我们在录制⽂件的时候，⼀定要保证stop()⽅法被调⽤，因为stop⾥⾯包含了写出⽂件头的操作，如果没有调⽤stop就会导致录制的⽂件损坏，⽆法被识
别或⽆法播放。
close()⽅法中包含stop()和release()⽅法
#### OpenCVFrameRecorder结构分析
OpenCVFrameRecorder的代码很简单，不到120⾏的代码，主要是基于opencv的videoWriter封装，流程与FrameRecorder的相同：
初始化和设置参数--->start()--->循环record(Frame frame)--->close()
在start中会初始化opencv的VideoWriter，VideoWriter是opencv中⽤来保存视频的模块，是⽐较简单的，可以对照参考opencv官⽅⽂档介绍：OpenCV: cv::VideoWriter Class
Reference。
#### OpenCVFrameRecorder的初始化和参数设置
OpenCVFrameRecorder初始化参数只有三个：filename（⽂件名称或者⽂件保存地址），imageWidth（图像宽度）, imageHeight（图像⾼度）。但是OpenCVFrameRecorder的
有作⽤的参数只有六个，其中pixelFormat和videoCodec这两个⽐较特殊。
pixelFormat：该参数并不像ffmpeg那样表示像素格式，这⾥只表示原⽣和灰度图，其中pixelFormat=1表示原⽣，pixelFormat=0表示灰度图。
videoCodec：这个参数，在opencv中对应的是fourCCCodec，所以编码这块的设置也要按照opencv的独特⽅式来设置，有关opencv的视频编码fourCC的编码集请参考：The
'MP4' Registration Authority和Video Codecs by FOURCC - fourcc.org这两个列表，由于列表数据较多，这⾥就不展示了。
filename（⽂件名称或者⽂件保存地址），imageWidth（图像宽度）, imageHeight（图像⾼度）和frameRate（帧率）这四个参数就不过多赘述了。
#### #OpenCVFrameRecorder开始录制start
调⽤start时其实就是初始化了opencv的VideoWriter。这个⽐较简单，传了上⾯那六个参数。
#### #OpenCVFrameRecorder录制record
在循环中把Frame放进record中就可以⼀直录制，内部是通过OpenCVFrameConverter把Frame转换成了Mat调⽤VideoWriter进⾏录制。
#### #OpenCVFrameRecorder销毁stop/close
不管是stop还是close，其实都是调⽤的release()⽅法， release则是调⽤了opencv的VideoWriter的release()，结构很简单。
