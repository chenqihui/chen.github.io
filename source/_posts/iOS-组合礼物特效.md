---
title: iOS 组合礼物特效
date: 2019-01-16 11:15:38
tags:
  - 二值化
  - 组合特效
categories:
  - iOS
  
---
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[QHGiftAnimationMan](https://github.com/chenqihui/QHGiftAnimationMan) 这个项目看看时间是16年2月，快三年前的工程了。最近重新 checkout 下来，将 Swift2.0 的语法根据提示修改为 Swift4.2。不得不说 Swift 每个版本的变化还是蛮多的，也增加了新功能。而该项目是作为 [iOS 动图播放 & 制作合集](http://chenqihui.github.io/2018/09/16/iOS-%E5%8A%A8%E5%9B%BE%E6%92%AD%E6%94%BE-%E5%88%B6%E4%BD%9C%E5%90%88%E9%9B%86/) 的续篇二，也是介绍直播间的礼物动图特效 —— 组合礼物特效。

<!-- more -->

#### 组合礼物特效的播放

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在赠送多个小礼物的情况下，很多 直播App 会对 小礼物的送礼 增加特效，从而提高用户和主播的体验，增加礼物的赠送反馈效果。目前比较常见的效果就是连击和本次要介绍的组合礼物特效。简单点说明就是把多个 小礼物 组合一个大礼物或者特定形状（文字）展现在直播间。再举个例子，就如送 999朵玫瑰，可以将多个小玫瑰拼成一个大玫瑰（大玫瑰每个点是由小玫瑰构成）。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;如下，玫瑰组合的笑脸特效，并通过随机小礼物的初始位置，通过移动组合而成。

<div align=center><img src="https://anakinpublicspace-1253727175.cos.ap-chengdu.myqcloud.com/blog/GiftGroupAnimation/xiao2.gif" width="165" height="180"/></div>

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;这种组合图形是比较简单，直接码代码记录需要移动的坐标即可，编写起来不难。可再简单，当量多上来，开发也很吃力。因此增加了配置的方式显示组合图形，也就是该项目主要功能。采用二值图（即0-1图），通过这种 0和1 表示该位置是否显示小礼物，如下的模板即可

```
0000000000000000
0000100000010000
0001010000101000
0001010000101000
0000000000000000
0000000000000000
0000000000000000
0001000000001000
0000100000010000
0000010000100000
0000001111000000
0000000000000000
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;这也是上面笑脸特效的二值图。

#### 图像的二值化

>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[图像二值化](https://baike.baidu.com/item/%E5%9B%BE%E5%83%8F%E4%BA%8C%E5%80%BC%E5%8C%96/1748870?fr=aladdin) 将256个亮度等级的灰度图像通过适当的阈值选取而获得仍然可以反映图像整体和局部特征的二值化图像。在数字图像处理中，二值图像占有非常重要的地位，首先，图像的二值化有利于图像的进一步处理，使图像变得简单，而且数据量减小，能凸显出感兴趣的目标的轮廓。其次，要进行二值图像的处理与分析，首先要把灰度图像二值化，得到二值化图像。
所有灰度大于或等于阈值的像素被判定为属于特定物体，其灰度值为255表示，否则这些像素点被排除在物体区域以外，灰度值为0，表示背景或者例外的物体区域。

#### 二值图的制作

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;上述的二值图可以由产品或者其他人完成（如写个中文或者英文等），不用修改代码就能显示特效，并且可以通过服务端进行分发下载，实现动态更新。

##### 大笨象效果

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;然而如果组合礼物特效变复杂的话，那么人工编辑难度就很大，并且不能保证好看，所以需要增加将图片改为灰度图，再将其二值数据保存下来，效果如下：

<table>
    <tr>
        <td ><center><img src="https://anakinpublicspace-1253727175.cos.ap-chengdu.myqcloud.com/blog/GiftGroupAnimation/d1.png">原始图 </center></td>
        <td ><center><img src="https://anakinpublicspace-1253727175.cos.ap-chengdu.myqcloud.com/blog/GiftGroupAnimation/d2.png">二值图</center></td>
    </tr>

    <tr>
        <td ><center><img src="https://anakinpublicspace-1253727175.cos.ap-chengdu.myqcloud.com/blog/GiftGroupAnimation/d3.png">组合图</center> </td>
    </tr>
</table>

大笨象的二值图文件：

<div align=center><img src="https://anakinpublicspace-1253727175.cos.ap-chengdu.myqcloud.com/blog/GiftGroupAnimation/bp.png"></div>

##### 转化的实现

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;这是将 UIImage 通过转换为 CGBitmapContext 之后，然后读取 data 的 RGB 值，计算是取 0 还是 1，这里简单进行求平均值来判断，而 space 是类似对图片的有损压缩，可能大家看过这个视频 [神奇！一张狗狗照片，裁碎竟变四张_OMG!](https://www.pearvideo.com/video_1302310)，将一个图形裁剪成一条条，然后间隔拼合后，会出现两张跟原来很像的图片，继续这样裁剪拼合会变成四张，这里就是按照这种原理，降低分辨率。需要这样做是由于分辨率太多，得出的矩阵会很大，会大大增加代码的计算，而实际上是不需要的，所以这种简单的有损压缩会让该功能更高效。而 space 值需要根据不同图片情况决定。

```
let ss = Int(max(space, 1))
        
var matrixBitmap: String = ""
let data = unsafeBitCast(c.data, to: UnsafeMutablePointer<UInt8>.self)
for y in 0..<height {
    if y%ss == 0 {
        var matrixLine: String = ""
        for x in 0..<width {
            let offset = 4 * (y * width + x)
            
//                    let alpha = data[offset]
            let red = data[offset+1]
            let green = data[offset+2]
            let blue = data[offset+3]
            
            let value = (CGFloat(red) + CGFloat(green) + CGFloat(blue)) / CGFloat(3.0) / CGFloat(255.0)
            
            if x%ss == 0 {
                if value < 0.5 {
                    matrixLine += "1"
                }
                else {
                    matrixLine += "0"
                }
            }
        }
        matrixBitmap += matrixLine + "\n"
    }
}
```


#### 后续

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;其实到这里基本的制作与播放都OK了，但是还是会发现不够。如果要这个组合礼物特效除了散列的组合动画，是否能做其他动画，比如说组合后运动、组合后局部运动等等（如组合成弓箭，然后箭是可以左右动，类似射箭的动作）。可以构想多张二值图来实现，这就像动图和视频的实现原理，一张张二值图组合的特效相当于一帧帧的画面。此 part 待续。

#### 注意

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;该项目可以说只是提供思路及思路的简单实现，是由于本人看到项目其他小伙伴开发实现组合礼物特效的时候，想到使用这样的二值图来配置实现会更通用，也更方便。但并没有实际运用到项目中，所以代码的高效性及可扩展性都还不足，因此该项目仅供参考学习。

参考

* [extract pixel from a CGImage](https://gist.github.com/jokester/948616a1b881451796d6)
* [【IOS】图片二值化和黑白(灰度)处理](https://www.jianshu.com/p/c962403cae65)
* [ColorMatcher_ios/PixelExtractor.swift](https://github.com/SimonVL01/ColorMatcher_ios/blob/58257833e5dffb2a8b35e027a5caa3963c0145cc/ColorMatcher/PixelExtractor.swift)
* [RGB 转换灰度图像原理](https://blog.csdn.net/x5675602/article/details/80014683)