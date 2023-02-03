---
title: 第六节 帧过滤器(FrameFilter)的原理与应用
date: 2022-02-09 15:31:34
tags: 流媒体
category: 死磕javaCV记录
top_img: https://s4.ax1x.com/2022/02/10/HYMSf0.jpg
cover: https://s4.ax1x.com/2022/02/10/HYMSf0.jpg
---
#### 前⾔
在此之前，我们分析了FrameGrabber和FrameRecorder，对于⾳视频、图⽚和流媒体的输⼊输出相信⼤家已经基本掌握和了然于⼼了。那么接下来的本章，主要讲解和分析
FrameFilter，让我们直接开始吧。
#### FrameFilter的介绍和结构
FrameFilter就是过滤⾳频和视频帧，并对⾳频和视频进⾏处理的⼀个帧处理器，⽤滤镜来描述可能更为贴切⼀点（但是由于FrameFilter还可以处理⾳频，所以我们还是使⽤“过滤
器”更合适些，虽然有可能引起歧义就是了），在采集到解码后的⾳视频源或者图像、⾳频后，对解码后的数据源进⾏加⼯的过程就是FrameFilter做的事情了。
#### FrameFilter处理流程
FrameFilter的⼀般调⽤处理流程
>初始化和设置解码后的数据--->start()--->循环start| push( Frame frame)--->Farme pull() |循环end--->结束时调⽤stop释放内存
> 
结合FrameGrabber和FrameRecorder后的FrameFilter处理流程
如下图所示 

![avatar](https://s4.ax1x.com/2022/02/09/HGQaVA.png)
FrameFilter 过滤器调⽤处理流程  

#### FrameFilter的⼦类
FrameFilter只有⼀个实现类就是FFmpegFrameFilter，所以本章主要分析FFmpegFrameFilter。
#### FFmpegFrameFilter剖析
#### FFmpegFrameFilter介绍
FFmpegFrameFilter本身就是FrameFilter的实现类， 结构基本相同，使⽤流程参考上⾯的流程结构图。
#### FFmpegFrameFilter代码结构剖析
FFmpegFrameFilter的初始化及参数
视频和⾳频混合过滤器初始化
> FFmpegFrameFilter(String videoFilters, String audioFilters, int imageWidth, int imageHeight, int audioChannels)
> 
视频过滤器初始化
> FFmpegFrameFilter(String filters, int imageWidth, int imageHeight)
> 
⾳频过滤器初始化
> FFmpegFrameFilter(String afilters, int audioChannels)
> 
参数
视频需要⾄少三个参数：videoFilter、imageWidth和imageHeight。也就是说必须保证这三个参数（视频过滤器，图像宽度，⾼度）都不能为空，且图像⾼度和宽度⼤⼩不能太离
谱。
⾳频过滤器需要⾄少两个参数：afilters 和 audioChannels。⾳频过滤器和⾳频通道
参考FFmpegFrameFilter源码：
> if (filters != null && imageWidth > 0 && imageHeight > 0) {undefined
startVideoUnsafe();
}
if (afilters != null && audioChannels > 0) {undefined
startAudioUnsafe();
}
> 
start⽅法
start()⽅法会对视频过滤器和⾳频过滤器进⾏判断，如果有视频过滤器和⾳频过滤器则就会初始化对应过滤器上下⽂。
push⽅法
> 可以同时送⼊视频图像像素和⾳频采样（Frame对象可以同时包含图像像素和⾳频采样数据）  
push(Frame frame)  
与上⾯⼀样，只不过可以设置视频图像像素格式  
push(Frame frame, int pixelFormat)  
上⾯两个⽅法都是这个⽅法的语法糖，也是⼀样同时⽀持送⼊视频和⾳频， 还⽀持更多参数，⽐如视频的宽度（width）、⾼度（height）、图像深度（depth），图像通道
（channels）、图像跨度（stride）、像素格式（pixelFormat），图像像素缓存数据  
pushImage(int width, int height, int depth, int channels, int stride, int pixelFormat, Buffer ... image)  
只送⼊⾳频采样数据，⾳频通道（audioChannels）, 采样率（sampleRate）, ⾳频编码格式（sampleFormat）和⾳频采样缓存  
pushSamples(int audioChannels, int sampleRate, int sampleFormat, Buffer ... samples)  
补充：关于图像深度（depth），图像深度是图像⾥⽤来表示⼀个像素点的颜⾊由⼏位构成，⽐如24位图像深度的1920x1080的⾼清图像，就表示这个⾼清图像它所占空间
是（1920x1080x24bit/8）kb也就是：1920x1080x3 kb  
关于图像通道（channels），channels*8=depth，也就说图像深度除8Bit就是图像通道数，图像跨度（stride）就是图像⼀⾏所占⻓度，⼀般⼤于等于图像的宽度。  
> 
pull⽅法
> 拉取经过视频和⾳频过滤器处理后的视频图像Frame，如果同时设置了⾳频和视频过滤器，会同时通过Frame返回，如果只设置了其中⼀个，则Frame中存放的也是只有其
中⼀个处理后的图像像素或者⾳频采样。
> 
pull()
只拉取经过视频过滤器处理后的图像像素
Frame pullImage()
只拉取经过⾳频过滤器处理后的⾳频采样
Frame pullSamples()
#### FFmpegFrameFilter的使⽤
使⽤示例参考javaCV开发详解之13：使⽤FFmpeg Filter过滤器处理⾳视频
