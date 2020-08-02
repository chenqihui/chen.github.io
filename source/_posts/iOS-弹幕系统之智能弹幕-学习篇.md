---
title: iOS 弹幕系统之智能弹幕 学习篇
date: 2019-06-12 12:00:00
tags:
  - layer
  - mask
  - UIImage
categories:
  - iOS
  
---

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;智能弹幕不挡脸效果就是使刷屏弹幕无法遮挡想看的内容。目前很多视频网站 & App 都已经实现，且还运用到直播上，提高了观看的体验。具体效果可以看看斗鱼等直播平台（记得确认该直播间是否有此功能）。接下来从客户端角度探讨如何实现智能弹幕，下面的效果的代码实现都在 -- 新弹幕系统 [chenqihui/QHDanmu2Demo](https://github.com/chenqihui/QHDanmu2Demo)，请同步工程了解。

* [虎牙推出弹幕黑科技：AI智能弹幕功能不挡脸](https://www.admin5.com/article/20181108/884048.shtml)

<!-- more -->

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;iOS客户端 首先想到是使用 mask 蒙层来实现。

#### CAGradientLayer

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;实现之后会发现它只能进行线性渐隐的绘制。

<div align=center><img src="https://anakinpublicspace-1253727175.cos.ap-chengdu.myqcloud.com/blog/Danmu/ai/G.png"/></div>

#### CAShapeLayer & UIBezierPath

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;这一次绘制出了镂空的多边形，此方案基本满足弹幕蒙层的需求，不足在于它无法实现边缘的渐隐效果。

<div align=center><img src="https://anakinpublicspace-1253727175.cos.ap-chengdu.myqcloud.com/blog/Danmu/ai/S.png"/></div>

#### UIView

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;只是采用 View 的话，比较容易绘制需要显示的区域（即整个 view 的 alpha 值为0，需要显示的位置添加 alpha 值为1的 view）。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;原本想着可以反选，即在整个 alpha 值为1的 view 上添加 alpha 值为0的 子view，从而实现镂空效果，再加入绘制四周阴影来实现。但事实上 父view 的 alpha 值为1后，在图层上已经是整个可见了，已无法再隐藏。

<div align=center><img src="https://anakinpublicspace-1253727175.cos.ap-chengdu.myqcloud.com/blog/Danmu/ai/v.png"/></div>

#### CGGradientRef 的圆半径方向渐变 UIImage

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;这其实使用了它来绘制一张 UIImage 来实现，它的不足在于其绘制多边形的能力有限。

<div align=center><img src="https://anakinpublicspace-1253727175.cos.ap-chengdu.myqcloud.com/blog/Danmu/ai/GR.png"/></div>

#### 带 Alpha 的 UIImage

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;从上述的描述可以看出，从苹果提供的 API 来同时实现有 渐隐 & 多边形镂空 效果暂时都没成功。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;因此得借助其他方式，其实关键还是生成带有 alpha 值的 UIImage。这里通过 Python 生成一个简单的带 alpha 值的圆形图片。按理也可以使用 PS 生成。

~~~python
import numpy as np
import math
import matplotlib.pyplot as plt

data = []

for i in range(400):
	print i+1
	item = []
	for j in range(400):
		weight = math.sqrt((math.pow((i-200), 2) + math.pow((j-200), 2)))
		if weight < 50:
			item.append([0,0,0,0])
		elif weight <= 100:
			ap = int(255.0/50*(weight-50))
			item.append([0,0,0,ap])
		else:
			item.append([0,0,0,255])
	data.append(item)
	
print 'start output img'	

img = np.array(data, 
		dtype=np.uint8)

plt.imsave('img.png', img)
~~~

<div align=center><img src="https://anakinpublicspace-1253727175.cos.ap-chengdu.myqcloud.com/blog/Danmu/ai/AI.png"/></div>

#### 自生成的带 Alpha 值的 UIImage

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;如果放在程序里面，不太好放很多已经生成的图片，所以还是需要 iOS 自己生成 alpha 值的 UIImage，生成的逻辑跟 Python 原理是一样的。

~~~objc
- (void)p_addCustomUIImage {
    UIView *v = [[UIView alloc] initWithFrame:_showMainV.bounds];
    v.backgroundColor = [UIColor whiteColor];
    
    UIGraphicsBeginImageContextWithOptions(v.bounds.size, NO, [UIScreen mainScreen].scale);
    CGContextTranslateCTM(UIGraphicsGetCurrentContext(),
                          v.frame.origin.x,
                          v.frame.origin.y);
    [v.layer renderInContext:UIGraphicsGetCurrentContext()];
    UIImage *image = UIGraphicsGetImageFromCurrentImageContext();
    UIGraphicsEndImageContext();
    
    CGImageRef originalMaskImage = [image CGImage];
    float width = CGImageGetWidth(originalMaskImage);
    float height = CGImageGetHeight(originalMaskImage);
    
    int strideLength = ceil(width);
    unsigned char * alphaData = calloc(strideLength * height, sizeof(unsigned char));
    CGContextRef alphaOnlyContext = CGBitmapContextCreate(alphaData,
                                                          width,
                                                          height,
                                                          8,
                                                          strideLength,
                                                          NULL,
                                                          kCGImageAlphaOnly);
    
    CGContextDrawImage(alphaOnlyContext, CGRectMake(0, 0, width, height), originalMaskImage);
    
    CGPoint c = CGPointMake(width/2, height/2);
    float r = 80;
    for (int y = 0; y < height; y++) {
        for (int x = 0; x < width; x++) {
            unsigned char val = alphaData[y*strideLength + x];
            float weight = sqrt(pow(x - c.x, 2) + pow(y - c.y, 2));
            if (weight < r) {
                val = 0;
            }
            else if (weight <= r * 2) {
                val = (int)(255.0/r*(weight-r));
            }
            else {
                val = 255;
            }
            alphaData[y*strideLength + x] = val;
        }
    }
    
    CGImageRef alphaMaskImage = CGBitmapContextCreateImage(alphaOnlyContext);
    UIImage *resultImage = [UIImage imageWithCGImage:alphaMaskImage];
    
    UIImageView *i = [[UIImageView alloc] initWithFrame:_showMainV.bounds];
    i.backgroundColor = [UIColor clearColor];
    i.image = resultImage;
    i.contentMode = UIViewContentModeScaleAspectFill;
    _showMainV.maskView = i;
}
~~~

<div align=center><img src="https://anakinpublicspace-1253727175.cos.ap-chengdu.myqcloud.com/blog/Danmu/ai/CI.png"/></div>

#### 总结

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;说了这么多，如果真的运用到智能弹幕上，虽然有些方案可以简单实现效果，只需要接入人形的坐标点即可。但是 MaskView 频繁使用的话，在 UIImage 的实时高效生成，会十分消耗性能，并且运用到直播与视频上的同步性也需要考虑。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;那重新回顾虎牙的介绍，有这一段说法

>虎牙AI技术负责人吴晓东向我们介绍了AI智能弹幕功能的特点：
>
>* 常规处理弹幕的做法是“离线(Offline)”和云上处理，需要面对的只是识别问题；而虎牙是针对直播进行实时端上处理；
>* 有的网站采用类似PS蒙版技术，采用人工方式为特定视频添加蒙版来模糊弹幕；而虎牙则采用人景分离技术，让人物与场景分离，让弹幕在人物之后，场景之前；
>* AI智能弹幕与传统直播弹幕相比，在几乎不增加带宽的前提下，把每帧的mask随视频流编码。而常规方法在视频点播的中则需要大量的流量来支撑弹幕传输;

* [看与说同享 虎牙采用人景分离技术打造不遮脸智能弹幕 - 51CTO.COM](http://ai.51cto.com/art/201811/586866.htm)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;采用人景分离，那如果通过 OpenGL，将人像再绘制一次。从而输出多一个 UIView，将弹幕View 放置在景物View 与 人物View 直接。那么此部分开发实现就移到了 播放器 部分，无须弹幕做而外的工作，因此智能弹幕的介绍与实现就等后面 播放器 部分再研究。

#### 补充

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;人脸识别的数据，在 rtmp 上可以通过 SEI 帧传输，即播放器解析 SEI 数据后进行人景分离。至于 SEI 可以参考下面 金山云 的文章：

* [FFmpeg从入门到精通——进阶篇，SEI那些事儿](https://mp.weixin.qq.com/s/BrSOPtDIOj5Lv1h1MLKCSA)
* [h264解码之自定义信息（SEI）](https://blog.csdn.net/y601500359/article/details/80943990)


