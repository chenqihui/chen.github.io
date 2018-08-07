---
title: MP4 格式了解下
date: 2018-08-07 01:32:02
tags:
  - Swift
  - MP4
  - MacOS
categories:
  - 音视频
  
---
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;继 [QHFlvParserMan](https://github.com/chenqihui/QHFlvParserMan) 解析 FLV 之后，本次增加了对 MP4 视频格式的解析支持。而通过上一篇 《[FLV 格式一谈](http://chenqihui.github.io/2018/06/20/FLV-%E6%A0%BC%E5%BC%8F%E4%B8%80%E8%B0%88/)》入门了音视频的知识后。可以通过对 MP4 了解，来更进一步学习音视频知识。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;MP4 多用于点播，它也是现在用的比较多的视频封装格式，它为了播放流式媒体的高质量视频而专门设计的，以求使用最少的数据获得最佳的图像质量。但是它的文件格式复杂，索引慢，时长较长的 MP4 视频在线播放时加载较慢。

<!-- more -->

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;MP4 是遵循 MPEG-4（ISO 14496-14）的官方容器格式定义的广义文件扩展名。它可以流媒体化并支持众多多媒体的内容（多音轨 (multiple audio)、视频流 (video)、字幕 (subtitlestreams)、图片(pictures)、可变桢率 (variable-framerates)、码率(bitrates)、采样率(samplerates)等）和高级内容 (advanced content)（官方称之为 “Richmedia” (超媒体) 或 “BIFS” (Binary Format for Scenes/二进制格式场景），类似 2D 和 3D 图形，动画、用户界面、类DVD 菜单，上述这些 AVI 搞不定的东西。

#### 工具

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;它解析了 MP4 文件的数据类型，各个类型包的大小、时间戳和简单信息，还有需要重点了解的 boxes 及 各个box 的内容。

<!-- 截图 -->

![](http://pacfu36li.bkt.clouddn.com/QHMP4ParserMan.png)

文章和工具的实现主要参考：

* [点播视频格式的选择 | SamirChen](http://www.samirchen.com/video-playback-format/)
* [leslie - wqyuwss的专栏](http://www.52rd.com/Blog/wqyuwss/559/)
* [mp4文件格式解析 - nigaopeng](https://www.cnblogs.com/ranson7zop/p/7889272.html)

#### 视频一谈 & MP4 注意问题

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;首先，了解到视频主要是 metadata 信息 + 音视频的 data，metadata 记录了视频的图像尺寸、编码格式、帧率、码率等信息。FLV 的 metadata 是保存在第一个视频和音频 tag 里。那 MP4 的 metadata 存放在 meta，它可以出现在视频的开头或者结尾，如果在结尾就会导致播放器只有下载完整个文件后才能成功解析并播放这个视频。又来介绍下 FFmpeg 了

```
ffmpeg -i bad.mp4 -movflags faststart good.mp4
```
将其重新编码，将 metadata 数据块转移到文件头部，保证播放器在线请求时能较快播放。

>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;此外，需要注意的是，在通过网络请求 MP4 视频文件进行边下载边播放时，由于接收到的每帧音视频数据是需要根据时间戳来决定何时送入解码器解码，以及何时显示，如果时间没到只能等待。尤其是当视频编码存在 B 帧时，解码时间戳 DTS 与显示时间戳顺序 PTS 通常不一致，这时候就会出现已经下载好的一帧数据可能还不能立即解码和显示，需要等待其依赖的数据下载完成和解码完成。这时在播放的过程中就可能因为网络抖动而出现播放卡顿。

#### MP4 文件结构和 Box 描述

##### 基本构成
 
|  | 构成 |  
| --- | --- |  
| MP4 | Boxes |  
| Box | Header + Data |  
| Header | Size(4bytes) + Type(4bytes) <br/> + [ (if Size == 0 ? version(8bits) + flags(24bits) : 无) ] |  
| Data | Container Boxes 或者 <br/> Box Size(4bytes) + Type(4bytes) + Version(1bytes) + Flay(3bytes) <br/> + [ EntryCount(4bytes) + EntryList(Nbytes) ] |  

![](http://pacfu36li.bkt.clouddn.com/mp4-structure.gif)

##### Box

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;由于 MP4 文件的 Box 类型非常多，**《[The 'MP4' Registration Authority](http://mp4ra.org/#/atoms)》** 列出了当前注册过的一些 Box 类型，这么多的 Box 类型要完全支持还是很复杂的。而 Demo 解析的 box 可以查看  [QHFlvParserMan 里 QHMP4 的 类文件](https://github.com/chenqihui/QHFlvParserMan) 的注释。

1、trak：Track Box，判断是两种类型的 trak：media track 和 hint track 哪一种
答：trak 里面的 mdia -> hdlr：Handler Reference Box，包含了表明该轨道类型的信息：Video Track、Audio Track 或者 Hint Track。

<!--![](http://pacfu36li.bkt.clouddn.com/mp4-structure-2.jpg)-->

#### 分析 MP4 文件信息  

##### 1、计算电影长度  
time scale 相当于定义了标准的1秒在这部电影里面的刻度是多少。   
方法1  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;从 mvhd - movie header atom 中找到 time scale 和 duration ，duration 除以time scale 即是 整部MP4 的长度。   
方法2  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;每个 trak -> mdia -> mdhd 里也会有 time scale 和 duration，通过其计算出 每个 track 的长度。  
Demo: 只有一个 video track 和 audio track，所以它们分别的长度都等于总长度。  
方法3  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;整部电影的duration = 每个帧的 duration 之和（从stts - Time-to-sample atoms中得出，即 sampleCount 乘以 sampleDuration），再除以 time scale（方法2 mdhd 读到的）

##### 2、计算电影图像宽度和高度  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;从tkhd – track header atom中找到宽度和高度即是（width & height）

##### 3、计算电影声音采样频率  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;从tkhd – track header atom中找出audio track的time scale即是声音的采样频率。

##### 4、计算视频帧率  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;首先计算出整部电影的duration，和帧的数目然后  
帧率 = 帧的数目 / 整部电影的duration  
Demo: 192 / 12.8 = 15

##### 5、计算电影的比特率  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;整部电影的尺寸除以长度，即是比特率（码率 kbps）  
Demo: stsz   
视频：1044840*8/12.8/1000=653.025kbit/s  
音频：208772*8/12.8/1000=130.4825kbit/s    

##### 6、查找sample   
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;当播放一部电影或者一个track的时候，对应的media handler必须能够正确的解析数据流，对一定的时间获取对应的媒体数据。如果是视频媒体， media handler可能会解析多个atom，才能找到给定时间的sample的大小和位置。

具体步骤如下：

1. 确定时间 t，相对于媒体时间坐标系统 t 乘以 time scale（from mdhd）
2. 检查 stts（time-to-sample atom）来确定给定时间的 sample序号  
3. 检查 stsc（sample-to-chunk atom）来发现对应该 sample 的 chunk  
	（在 stsc 的 entrylist 里面：用下一个 entry 的 firstChunk 减去本次的 firstChunk，就得到了这组 chunk 的个数。最后一个 entry 结构体则表明从该firstChunk 到最后一个 chunk，每个 chunk 都有 samplsPerChunk 个 sample。  
	即：firstChunk(N+1) - firstChunk(N) = chunkCount）
4. 从 stco（chunk offset atom）中提取该 chunk 的偏移量  
5. 利用 stsz（sample size atom）找到 sample 在 chunk 内的偏移量和 sample 的大小

Demo：如果要找第1秒的视频数据，过程如下：

1. 第1秒的视频数据相对于此视频（timescale = 15360）的时间为 15360
2. 检查 stts（entryInfo = [["sampleCount": 192, "sampleDuration": 1024]]），得出这里 sample 的 duration 都是 1024，从而得出需要寻找第15360/1024 = 15 + 1 = 16个sample
3. 检查 stsc（entryInfo = [["firstChunk": 1, "samplesPerChunk": 2, "sampleDescriptionID": 1], ["firstChunk": 2, "samplesPerChunk": 1, "sampleDescriptionID": 1]]），为 2 列，则，第二个firstChunk减第一个 = 1，说明第一列是 1个chunk，包含 2个sample，然后通过 stco（entryCount = 191）知道 chunk 的总数是 191，也就是说 第二列 190 个chunk，各包含 1个sample，而 总smaple为192 也等于 stts（"sampleCount": 192）。这样就得出在 第15（16个sample -> 2 + 14 sample -> 1 + 14 chunk -> 15 chunk）个chunk。
4. 检查 stco（chunkOffset）找到 第15个chunk 的偏移量是 136470
5. 由于 第16个sample 是 第15个trunk 的 第一个sample，所以不用检查 sample size atom，chunk 的偏移量即是该 sample 的偏移量 136470。如果是这个 chunk 的第二个 sample，则从 sample size atom 中找到该 chunk 的前一个 sample 的大小，然后加上偏移量即可得到实际位置。
6. 得到位置后，即可取出相应数据进行解码，播放

##### 7、查找关键帧  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;查找过程与查找sample的过程非常类似，只是需要利用sync sample atom来确定key frame的sample序号  

1. 确定给定时间的sample序号 
2. 检查sync sample atom来发现这个sample序号之后的key frame
3. 检查sample-to-chunk atom来发现对应该sample的chunk
4. 从chunk offset atom中提取该trunk的偏移量
5. 利用sample size atom找到sample在trunk内的偏移量和sample的大小  

##### 8、Seeking
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;那么我们如果对 MP4 文件做 Seek 操作呢？步骤如下：

1. 根据 mvhd(Movie Header Box) 中的 timescale 确定给定时间 t 在媒体时间坐标系统中的位置 p，即 t * timescale。
2. 根据 stts(Decoding Time to Sample Box) 查询指定 track 在时间坐标 p 之前的第一个 sample number。
3. 根据 stss(Sync Sample Box) 查询 sample number 前的第一个关键帧 sync sample。
4. 根据 stsc(Sample To Chunk Box) 查询对应 sync sample 的 chunk。
5. 根据 stco(Chunk Offset Box) 查询该 chunk 的在文件中的偏移量。
6. 根据 stsc(Sample To Chunk Box) 和 stsz(Sample Size Boxes) 的信息计算出该 chunk 中需要读取的 sample 数据，即完成 seek。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;补充：[Random access](http://www.52rd.com/Blog/Detail_RD.Blog_wqyuwss_7936.html) 

#### 补充 

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;MP4文件、MOV文件和3GP文件，这三种媒体文件格式采用了相同的封装格式。[Mp4编码全介绍 - AlphaJay](https://my.oschina.net/alphajay/blog/4276)

##### MOV  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**苹果官方内容：[Overview of QTFF](https://developer.apple.com/library/archive/documentation/QuickTime/QTFF/QTFFChap1/qtff1.html)**。它是1998年由苹果公司开发的一款视频格式，人们通常定义它为QuickTime播放器格式。它采用MPEG 4解码器进行压缩并兼容不同的轨道以便于储存电影和其他视频文件。它每一条轨道通常都要和至少一个或多个不同的编解码器进行编码。与其他视频格式相比，MOV毕竟是一种有损压缩文件，它之所以被广泛使用，主要还是因为它出色的兼容性能。它不仅可以与Macintosh平台兼容，也可以在Windows PC上运行的非常顺畅的。甚至是一些DVCPRO（一种高清的DV格式）文件也可以播放MOV视频。


#### 参考：

* [点播视频格式的选择 | SamirChen](http://www.samirchen.com/video-playback-format/)
* [leslie - wqyuwss的专栏](http://www.52rd.com/Blog/wqyuwss/559/)
* [mp4文件格式解析 - nigaopeng](https://www.cnblogs.com/ranson7zop/p/7889272.html)
* [MP4容器格式](http://zzqhost.github.io/hostwiki/%E5%A4%9A%E5%AA%92%E4%BD%93%E7%9B%B8%E5%85%B3_MP4%E5%AE%B9%E5%99%A8%E6%A0%BC%E5%BC%8F.html#toc_1.1)
* [Mp4编码全介绍 - AlphaJay](https://my.oschina.net/alphajay/blog/4276)
* [播放器适配经验总结——IOS](https://blog.csdn.net/luansxx/article/details/7721282)  
* [MP4文件格式详解——元数据moov（一）mvhd box](https://blog.csdn.net/pirateleo/article/details/7590056/)
* [MP4解析](https://blog.csdn.net/koozxcv/article/details/50502034)
* [stagefright之MPEG4Extractor（二）(stts,stsc,stco)](https://blog.csdn.net/tung214/article/details/30492895)

