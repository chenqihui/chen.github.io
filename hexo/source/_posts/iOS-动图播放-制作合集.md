---
title: iOS 动图播放 & 制作合集
date: 2018-09-16 22:49:15
tags:
  - Gif
  - WebP
  - Svga
  - LivePhoto
categories:
  - 动图
  - 音视频
  
---
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;今天聊到动图，它们在 App 上都很常见，如聊天的表情，直播的送礼 & 座驾特效，H5 上的活动图，视频的录制等等。而动图有很多种格式，常用的 帧动画、Gif、WebP、Apng、Svga。由于最近项目加入 Svga，因此顺便学习下其他动图，也把之前记录的 LivePhoto 一起总结进来，做一次复习。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;本篇可以说是一篇播放 & 制作的合集。大家将会看到 YYImage，Svga-Player，iSparta 转换等开源库 & 工具。而主要说明的动图格式是 Gif、WebP、Svga、LivePhoto。除了 LivePhoto，其它动图的的播放在 iOS 端的大概流程都是通过 ImageIO 绘制出 UImage，再通过 CADisplayer 进行逐帧播放，只是解码库不同，如 WebP 是依靠 Google 提供的库（因为编码不一样）。也正由于他们是逐帧播放的，所以可以将每帧数据解码后再重新编码成其他格式的动态。最为关键的还是编码上，从而形成各自的优势，就目前项目的动图还是比较多使用 WebP，而 Svga 目前使用上似乎更好，特别是在直播的大礼物特效上。

<!-- more -->

#### 使用记录

| 时间点 | 原因 | 结果 |  
| :---: | --- | --- |
| 1 | App 按钮太素 | 加入 png 来显示按钮 |
| 2 | 静态图片看厌了 | 将 pngs 连续播放 |
| 3 | 多端无法共用 | 使用 Gif |
| 4 | 有锯齿，且体积很大 | 换成 WebP  |
| 5 | 动图体积得再小 | 换成 Svga |
| 6 |  |  |  |

#### 前文

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;可以先通过 YYImage 作者 ibireme 博文的前半部分来了解 图片历史后，接着阅读本章后续内容，最后再回来看后半部分的知识点。

* [移动端图片格式调研 | Garan no dou](https://blog.ibireme.com/2015/11/02/mobile_image_benchmark/)

本篇 Demo：  

* [chenqihui/QHAnimationImageDemo: 动图](https://github.com/chenqihui/QHAnimationImageDemo)

#### 正文

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;动图都是播放帧图片，即解码出来的一张张图片就是需要播放的，因此它们的关键数据就是 Image & Duration，

* [图片处理：Image I/O 学习笔记](https://www.jianshu.com/p/4dcd6e4bdbf0)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;一般的图片在播放过程都经历: **解码 -> 绘制**

##### YYImage

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;YYImage 使用了 CADisplayLink & NSOperationQueue 来进行动画的逐帧绘制。它也几乎支持列出的大部分动图格式的播放。其中除了 WebP & APNG 两个，其余支持的图片格式都使用了 ImageIO 来进行解码。

参考

* [ibireme/YYImage](https://github.com/ibireme/YYImage)
* [YYImage 设计思路，实现细节剖析](https://lision.me/yyimage/)
* [iOS 处理图片的一些小 Tip | Garan no dou](https://blog.ibireme.com/2015/11/02/ios_image_tips/)

##### GIF

```objc
// 从 gif 里解码出单张 image
CGImageRef imageRef = CGImageSourceCreateImageAtIndex(_source, index, (CFDictionaryRef)@{(id)kCGImageSourceShouldCache:@(YES)});
UIImage *image = [UIImage imageWithCGImage:imageRef scale:_scale orientation:_orientation];
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;制作 Gif 的函数可以查看 [chenqihui/QHAnimationImageDemo](https://github.com/chenqihui/QHAnimationImageDemo)

参考

* [谈谈 GIF 格式](https://zhuanlan.zhihu.com/p/22590949)
* [iOS 的 GIF 动画效果实现](https://juejin.im/entry/58a3ba8c61ff4b006c3c3c8a)
* [iOS-加载gif的四种方式](https://www.jianshu.com/p/da89d102f3a0)

##### APNG

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;由于项目并没有使用，所以 Demo 没有添加，但在 YYImage 源码里可以看到通过 

```
static uint8_t *yy_png_copy_frame_data_at_index(const uint8_t *data,
                                                const yy_png_info *info,
                                                const uint32_t index,
                                                uint32_t *size)
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;获取 bytes 之后，再有 ImageIO 转为 UIImage，也就是说上述函数是完整的 APNG 解码过程了。

##### WebP

```
WEBP_EXTERN(int) WebPDemuxGetFrame(
    const WebPDemuxer* dmux, int frame_number, WebPIterator* iter);
    
WEBP_EXTERN(VP8StatusCode) WebPDecode(const uint8_t* data, size_t data_size,
                                      WebPDecoderConfig* config);
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;通过 WebP 库将数据解码出来，最后也是通过 CGImageCreate 来生成 UIImage。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;调用 WebP 库的方法可以参考 YYImageCoder ，或者搜索一下，如 [iOS-WebP/UIImage+WebP.m](https://github.com/seanooi/iOS-WebP/blob/master/iOS-WebP/UIImage%2BWebP.m) ，原理是一样的。

参考：

* [A new image format for the Web](https://developers.google.com/speed/webp/)
* [WebP（整理）](https://www.jianshu.com/p/12c4171c45c7)


##### Svga

>SVGA 是一种动画格式  
>SVGA 类似于 Dragonbones / CreateJS

* [yyued/SVGAPlayer-iOS](https://github.com/yyued/SVGAPlayer-iOS)
* [yyued/SVGA-Format](https://github.com/yyued/SVGA-Format)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;调用来的 **绘制：SVGAPlayer & 解码：SVGAParser**

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;集成它的时候，搜索下估计都会看到这两篇

* [直播App 中 Android 酷炫礼物动画实现方案（上篇）](https://mp.weixin.qq.com/s?__biz=MzI0ODQ5MTI3Nw==&mid=2247484070&idx=1&sn=d7848af31c2c670fc19fa3152a6549c6&scene=21#wechat_redirect)
* [直播App 中 Android 酷炫礼物动画实现方案（下篇）](https://mp.weixin.qq.com/s?__biz=MzI0ODQ5MTI3Nw==&mid=2247484074&idx=1&sn=4d4bd4147839e9cdc3b3d418da52e589&scene=21#wechat_redirect)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;它的制作不是序列帧，而是将动画切割一个个小图片，然后对其进行独立播放，类似游戏开发：保存各部分切图组成的纹理集和动画数据，只需要极少的原画，便可完成千变万化的动作动画组合。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;下面就是 Svga 使用的对象 SVGAVideoEntity，其中 images & sprites，对应的就是 图片元素 & 动画数据。

```objc
@interface SVGAVideoEntity : NSObject

@property (nonatomic, readonly) CGSize videoSize;
@property (nonatomic, readonly) int FPS;
@property (nonatomic, readonly) int frames;
@property (nonatomic, readonly) NSDictionary<NSString *, UIImage *> *images;
@property (nonatomic, readonly) NSArray<SVGAVideoSpriteEntity *> *sprites;
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;由于这样的模式，它提供了可以替换某一图片元素的 API，这也方便在动图中通过占位图片，来加入其它自定义的图片元素。相比其它动图需要多帧获取 & 替换才能达到的效果，其确实更方便。

##### LivePhoto

>Live Photo 是由一段3秒的视频 + 一张图片构成的。  
>原生 LivePhoto 视频采集的时间区间是由 按键后1.5s + 按键前1.5s 构成的。 
> 
>>[-1.5s ~ 0s, 拍摄瞬间,0s ~ 1.5s]。
>  
>复制代码最后合成 LivePhoto 展示的照片，取得是相机采集后 3s 片段中的中央那一帧。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Demo 采用 Mov 转换 LivePhoto 的方式，这也是目前 短视频App 生成 LivePhoto 壁纸的方式。它的播放器使用的是 PHLivePhoto，同时保存在相册就可以做为手机的壁纸，详细请查看 [chenqihui/QHAnimationImageDemo](https://github.com/chenqihui/QHAnimationImageDemo)，里面保存相册的代码注释了。

* [genadyo/LivePhotoDemo](https://github.com/genadyo/LivePhotoDemo)
* [三步为你的 App 集成 LivePhoto 功能](https://www.jianshu.com/p/83eb0ac3d9c0)

##### 纸上电影

<div align=center><img width = '320' height ='250' src ="https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1537249274742&di=d3c3bffb703b388fce26ace80b12b300&imgtype=0&src=http%3A%2F%2Fimg008.hc360.cn%2Fg1%2FM06%2F54%2FBA%2FwKhQL1Mhbd-EQByMAAAAANjfEiE062.jpg"/></div>

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;它的制作可以看看接下来的视频（杨超越喔，哈哈），只需一张图片 + 屏幕片，然后按方向拖动 屏幕片 即可出现动画。它算是模拟了屏幕的成像，跟走马灯其实是一样的，只是它采用一张原图，通过栅栏图片的遮罩显示不同的图片位置。至于在 iOS 实现，目前还在研究ing。

* [看着她，我许下一个愿 密码信成像原理，拿走拿走不客气](https://www.iqiyi.com/w_19s2cx01w9.html)
* [新奇特玩具纸上电影院产品简介](http://video.tudou.com/v/XMjIxMzYzMzc2NA==.html?__fr=oldtd)

<!--<iframe width="560" height="315" src="https://www.iqiyi.com/w_19s2cx01w9.html" frameborder="0" allowfullscreen></iframe>-->

参考：

* [纸上电影](https://baike.baidu.com/item/%E7%BA%B8%E4%B8%8A%E7%94%B5%E5%BD%B1/8408505#3)
* [视觉暂留](https://baike.baidu.com/item/%E8%A7%86%E8%A7%89%E6%9A%82%E7%95%99/5125149?fr=aladdin)
* [这些神奇的“视觉暂留”动画，每一幅都让人拍案叫绝！](https://baike.baidu.com/tashuo/browse/content?id=08b8cb850d53c863d33764e6&lemmaId=&fromLemmaModule=list)

#### 转换

##### pngs 转 GIF、Apng、WebP 工具

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;转换在参考文中都十分详细，大部分测试也是有用的，因此直接查看即可。

参考：

* [GIF与APNG，解决GIF锯齿问题](http://www.ui.cn/detail/34100.html)
* [iSparta/iSparta](https://github.com/iSparta/iSparta)


* [GIF/MOV/Live Photo](https://zhuanlan.zhihu.com/p/24077346)
* [NSRare/NSGIF](https://github.com/NSRare/NSGIF)

#### PC 制作 WebP

* [Downloading and Installing WebP](https://developers.google.com/speed/webp/download)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;先下载 libwebp，里面提供 .a & .h外，还提供 bin，这样即可新建项目使用，也可以命令使用，这里讲讲使用 bin。  

##### 环境变量配置 & PATH

1、配置单用户全局变量（重启后配置失效）

```
// /Users/用户名(e.g.chen)/.bash_profile
export WEBP=/Users/chen/Downloads/libwebp-1.0.0-mac-10.13/bin
export PATH=$PATH:$WEBP

// 更新配置
source .bash_profile
```

2、全局配置（永久有效）

```
// /etc/profile 
（同上）

// 更新配置
source /etc/profile
```

##### pngs2webp 

1、单张转 无动画webp，再合并成 动画webp

* cwebp
* webpmux

2、看 iSparta 里面是先壮 apng，再转 webp 

* 1、pngs2apng
* 2、apng2webp

3、Google 提供了将多张图片转为 webp，并且多张图片格式可以不同

* img2webp

参考：

* [png 序列帧转换为动态 webp --使用 google 提供工具及命令行脚本](https://blog.csdn.net/wyg1230/article/details/81045537)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;除了以上三种常用的步骤，当然还有其他，如先 GIF，再 webP。

#### 总结

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;以上的动图都有各自的优缺点，按需使用。WebP 是使用了视频的 VP8 编码格式。而 Svga 采用的方式在 体积 & 解码 上从理论与实际使用看都更好，越大的图会越明显。Svga 不仅对文件进行 protobuf % zlib 的压缩, 关键还在 PC 工具上，将 设计师 的动画原型转出 Svga 需要的 Bitmap & Frame, 这种类似游戏的动画模式资源，可以说很 cool 哈。当然看完其实也发现，除了 Svga，其他格式是可以在 iOS 端进行相互转换，这也是常常在客户端看到的 录制 Gif、截屏、录屏、制作 LivePhoto 壁纸、使用 WebP 也可以在客户端制作等等。



