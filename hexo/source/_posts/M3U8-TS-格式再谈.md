---
title: M3U8 & TS 格式再谈
date: 2018-10-06 16:54:35
tags:
  - Swift
  - M3U8
  - TS
  - MacOS
categories:
  - 音视频
  
---

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;我们已经了解到 m3u8 其实是一个文件，是 HLS 协议实现点播 & 直播的，还有 TS 文件。这里还有一篇讲解 AVPlayer 的，等编辑完再更新地址。

#### 理论

>HTTP Live Streaming (HLS) sends audio and video over HTTP from an ordinary web server for playback on iOS-based devices—including iPhone, iPad, iPod touch, and Apple TV—and on desktop computers (macOS). Using the same protocol that powers the web, HLS deploys content using ordinary web servers and content delivery networks. HLS is designed for reliability and dynamically adapts to network conditions by optimizing playback for the available speed of wired and wireless connections.
>
>HLS supports the following:
>
>* Live broadcasts and prerecorded content (video on demand, or VOD)
>* Multiple alternate streams at different bit rates
>* Intelligent switching of streams in response to network bandwidth changes
>* Media encryption and user authentication

<!-- more -->

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;HLS 的了解主要参考苹果文档，包括其协议、格式、部署等

* [HTTP Live Streaming (HLS) - Apple Developer](https://developer.apple.com/streaming/)
* [HTTP Live Streaming Overview](https://developer.apple.com/library/archive/documentation/NetworkingInternet/Conceptual/StreamingMediaGuide/Introduction/Introduction.html#//apple_ref/doc/uid/TP40008332)

* [HLS 实现点播和直播时，M3U8 文件的不同](https://blog.csdn.net/xiaojun111111/article/details/52102454)

#### 制作

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;先了解下如何制作 m3u8，有以下几种常用的方式

##### ffmpeg

```
# 1、先用 ffmpeg 把 abc.mp4 文件转换为 abc.ts 文件：
ffmpeg -y -i abc.mp4 -vcodec copy -acodec copy -vbsf h264_mp4toannexb abc.ts

# 2、再用 ffmpeg 把 abc.ts 文件切片并生成 playlist.m3u8 文件，5秒一个切片：
ffmpeg -i abc.ts -c copy -map 0 -f segment -segment_list playlist.m3u8 -segment_time 5 abc%03d.ts
```

```
// 1
ffmpeg version 4.0 Copyright (c) 2000-2018 the FFmpeg developers
  built with Apple LLVM version 9.1.0 (clang-902.0.39.1)
  configuration: --prefix=/usr/local/Cellar/ffmpeg/4.0 --enable-shared --enable-pthreads --enable-version3 --enable-hardcoded-tables --enable-avresample --cc=clang --host-cflags= --host-ldflags= --enable-gpl --enable-libmp3lame --enable-libx264 --enable-libxvid --enable-opencl --enable-videotoolbox --disable-lzma
  libavutil      56. 14.100 / 56. 14.100
  libavcodec     58. 18.100 / 58. 18.100
  libavformat    58. 12.100 / 58. 12.100
  libavdevice    58.  3.100 / 58.  3.100
  libavfilter     7. 16.100 /  7. 16.100
  libavresample   4.  0.  0 /  4.  0.  0
  libswscale      5.  1.100 /  5.  1.100
  libswresample   3.  1.100 /  3.  1.100
  libpostproc    55.  1.100 / 55.  1.100
Input #0, mov,mp4,m4a,3gp,3g2,mj2, from 'hehe.mp4':
  Metadata:
    major_brand     : isom
    minor_version   : 512
    compatible_brands: isomiso2avc1mp41
    encoder         : Lavf56.40.101
  Duration: 00:04:03.53, start: 0.000000, bitrate: 837 kb/s
    Stream #0:0(und): Video: h264 (High) (avc1 / 0x31637661), yuv420p, 960x544, 768 kb/s, 25 fps, 25 tbr, 12800 tbn, 50 tbc (default)
    Metadata:
      handler_name    : VideoHandler
    Stream #0:1(eng): Audio: aac (HE-AAC) (mp4a / 0x6134706D), 44100 Hz, stereo, fltp, 64 kb/s (default)
    Metadata:
      handler_name    : SoundHandler
Output #0, mpegts, to 'abc.ts':
  Metadata:
    major_brand     : isom
    minor_version   : 512
    compatible_brands: isomiso2avc1mp41
    encoder         : Lavf58.12.100
    Stream #0:0(und): Video: h264 (High) (avc1 / 0x31637661), yuv420p, 960x544, q=2-31, 768 kb/s, 25 fps, 25 tbr, 90k tbn, 12800 tbc (default)
    Metadata:
      handler_name    : VideoHandler
    Stream #0:1(eng): Audio: aac (HE-AAC) (mp4a / 0x6134706D), 44100 Hz, stereo, fltp, 64 kb/s (default)
    Metadata:
      handler_name    : SoundHandler
Stream mapping:
  Stream #0:0 -> #0:0 (copy)
  Stream #0:1 -> #0:1 (copy)
Press [q] to stop, [?] for help
frame= 6088 fps=0.0 q=-1.0 Lsize=   27544kB time=00:04:03.48 bitrate= 926.7kbits/s speed=1.65e+03x
video:22847kB audio:1903kB subtitle:0kB other streams:0kB global headers:0kB muxing overhead: 11.286891%
    
// 2
ffmpeg version 4.0 Copyright (c) 2000-2018 the FFmpeg developers
  built with Apple LLVM version 9.1.0 (clang-902.0.39.1)
  configuration: --prefix=/usr/local/Cellar/ffmpeg/4.0 --enable-shared --enable-pthreads --enable-version3 --enable-hardcoded-tables --enable-avresample --cc=clang --host-cflags= --host-ldflags= --enable-gpl --enable-libmp3lame --enable-libx264 --enable-libxvid --enable-opencl --enable-videotoolbox --disable-lzma
  libavutil      56. 14.100 / 56. 14.100
  libavcodec     58. 18.100 / 58. 18.100
  libavformat    58. 12.100 / 58. 12.100
  libavdevice    58.  3.100 / 58.  3.100
  libavfilter     7. 16.100 /  7. 16.100
  libavresample   4.  0.  0 /  4.  0.  0
  libswscale      5.  1.100 /  5.  1.100
  libswresample   3.  1.100 /  3.  1.100
  libpostproc    55.  1.100 / 55.  1.100
Input #0, mpegts, from 'abc.ts':
  Duration: 00:04:03.52, start: 1.480000, bitrate: 926 kb/s
  Program 1
    Metadata:
      service_name    : Service01
      service_provider: FFmpeg
    Stream #0:0[0x100]: Video: h264 (High) ([27][0][0][0] / 0x001B), yuv420p(progressive), 960x544, 25 fps, 25 tbr, 90k tbn, 50 tbc
    Stream #0:1[0x101](eng): Audio: aac (HE-AAC) ([15][0][0][0] / 0x000F), 44100 Hz, stereo, fltp, 63 kb/s
[segment @ 0x7ffbae829a00] Opening 'abc000.ts' for writing
Output #0, segment, to 'abc%03d.ts':
  Metadata:
    encoder         : Lavf58.12.100
    Stream #0:0: Video: h264 (High) ([27][0][0][0] / 0x001B), yuv420p(progressive), 960x544, q=2-31, 25 fps, 25 tbr, 90k tbn, 25 tbc
    Stream #0:1(eng): Audio: aac (HE-AAC) ([15][0][0][0] / 0x000F), 44100 Hz, stereo, fltp, 63 kb/s
Stream mapping:
  Stream #0:0 -> #0:0 (copy)
  Stream #0:1 -> #0:1 (copy)
Press [q] to stop, [?] for help
[segment @ 0x7ffbae829a00] Opening 'playlist.m3u8.tmp' for writing
[segment @ 0x7ffbae829a00] Opening 'abc001.ts' for writing
[segment @ 0x7ffbae829a00] Opening 'playlist.m3u8.tmp' for writing
[segment @ 0x7ffbae829a00] Opening 'abc002.ts' for writing
[segment @ 0x7ffbae829a00] Opening 'playlist.m3u8.tmp' for writing
[segment @ 0x7ffbae829a00] Opening 'abc003.ts' for writing
[segment @ 0x7ffbae829a00] Opening 'playlist.m3u8.tmp' for writing
frame= 6088 fps=0.0 q=-1.0 Lsize=N/A time=00:04:03.48 bitrate=N/A speed=1.35e+03x
video:22883kB audio:1938kB subtitle:0kB other streams:0kB global headers:0kB muxing overhead: unknown
```

参考：

* [ffmpeg对mp4文件进行ts切片并生成m3u8文件](https://blog.csdn.net/avsuper/article/details/72910907)
* [利用ffmpeg将MP4文件切成ts和m3u8](https://www.cnblogs.com/ChouDanDan/p/5566335.html)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;FFmpeg 切片  

* [FFmpeg Formats Documentation](http://ffmpeg.org/ffmpeg-formats.html#segment_002c-stream_005fsegment_002c-ssegment)


##### mediastreamsegmenter


```
which mediastreamsegmenter
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;通过如上校验是否安装 mediastreamsegmenter，如果没有输出 '/usr/local/bin/mediastreamsegmenter'，则可以通过 [More Downloads for Apple Developers](https://developer.apple.com/download/more/) 搜索 & 下载 & 安装（这里本人安装时候要求系统为 10.13.4 以上）

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;然后使用 [VLC media player](http://www.videolan.org/vlc/) + mediastreamsegmenter 进行切割，但是进行切割时候都出现 error，如下

```
Sep 20 2018 11:47:37.447: audio pid set at c8
Sep 20 2018 11:47:37.447: video pid set at 64
Sep 20 2018 11:47:39.923: error in pid c8 (audio) - cc value should be 11 is 1
Sep 20 2018 11:47:39.923: error in pid 64 (video) - cc value should be 1 is 2
Sep 20 2018 11:47:39.923: error with dts last 20870876 now 65038674
Sep 20 2018 11:47:39.924: average bit rate is  0.00 bits/sec - max file bit rate is  0.00 bits/sec
Abort trap: 6
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;还需要后续测试，而参考文章里面有使用另一个工具 **XAMPP** 来推 HLS 流。

参考

* [HLS（HTTP Live Streaming）视频直播技术实战 - 简书](https://www.jianshu.com/p/7902a54c25f0)
* [Mac OS环境下流媒体分割工具 mediastreamsegmenter 的简单使用](https://blog.csdn.net/musou_ldns/article/details/7493346)


##### mediafilesegmenter

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;同样来自工具 mediastreamsegmenter，下载参考上述 [More Downloads for Apple Developers](https://developer.apple.com/download/more/)。

```
mediafilesegmenter -B TEST -i test.m3u8 -t 10 -f ./video  ~/Movies/test.mp4

-B 切片名
-i m3u8文件
-t 时间间隔
-f ts切片存放位置
```

```
Sep 20 2018 11:59:30.144: Processing file /Users/chen/Desktop/mm/good.mp4
Sep 20 2018 11:59:30.160: track 2 of /Users/chen/Desktop/mm/good.mp4 contains edit list that the media doesn't start at beginning; this may cause problems in lip sync
Sep 20 2018 11:59:30.165: Finalized /Users/chen/Desktop/mm/stream/hehe/TEST0.ts
Sep 20 2018 11:59:30.166: Wrote full file size 557984 to /Users/chen/Desktop/mm/stream/hehe/TEST0.ts
Sep 20 2018 11:59:30.166: segment bitrate 892.77 kbits/sec is new max
Sep 20 2018 11:59:30.170: Finalized /Users/chen/Desktop/mm/stream/hehe/TEST1.ts
Sep 20 2018 11:59:30.172: Wrote full file size 489928 to /Users/chen/Desktop/mm/stream/hehe/TEST1.ts
Sep 20 2018 11:59:30.172: segment does not contain sync frame
Sep 20 2018 11:59:30.175: Finalized /Users/chen/Desktop/mm/stream/hehe/TEST2.ts
Sep 20 2018 11:59:30.175: Wrote full file size 241768 to /Users/chen/Desktop/mm/stream/hehe/TEST2.ts
Sep 20 2018 11:59:30.175: segment does not contain sync frame
Sep 20 2018 11:59:30.175: average bit rate is 814.53 kbits/sec - max file bit rate is 892.77 kbits/sec
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;此命令还为新增与媒体文件同名的 plist & iframe_index.m3u8，plist。mediafilesegmenter 与 mediastreamsegmenter 区别主要是一个是针对文件，一个针对的是 UDP 的实时 HLS 流。

>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Media Stream Segmenter（mediastreamsegmenter）通过UDP网络连接或 stdin 接收 MPEG-2 传输流，并将其分成一系列持续时间相同的小媒体段。然后，它会创建一个索引文件，其中包含对各个媒体段的引用。
>
>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;媒体文件分段器（mediafilesegmenter）将 MOV，MP4，M4V，M4A 或 MP3 文件分成媒体段并创建索引文件。可以使用几乎任何Web服务器基础结构部署索引文件和媒体段，以便流式传输到 iOS，macOS 和 tvOS。

参考：

* [SJIE · CodeIt!](http://www.lnmcc.net/page21/)
* [Mac OS环境下媒体文件分割工具mediafilesegmenter的简单使用](http://xpp888.blog.163.com/blog/static/14730596320126211424075/)


##### GStreamer & Gstreamill

>Gstreamill 是一个支持 hls 输出的，基于 gstreamer 的实时编码器。

#### 播放测试

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;把切割成功的 m3u8 & ts 源文件放在 Mac 的 WebServer -> Documents 目录下，并开启 Apache 后，即可使用 Safari 或者 QuickTime（使用“文件”>“打开位置...”或“⌘L”打开URL）等播放器来测试播放。 


#### 解析

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;这里主要解析 m3u8 描述文件 & ts 媒体文件，mp4 暂忽略

![TS](https://anakinpublicspace-1253727175.cos.ap-chengdu.myqcloud.com/blog/ts.png)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;源码 Demo [QHFlvParserMan](https://github.com/chenqihui/QHFlvParserMan) 是采用 Swift 实现的，内容可结合此工程进行了解。

##### M3U8 标签

这个在 AVPlayer 那里也有简单介绍过

```
// ffmpeg 输出的
#EXTM3U
#EXT-X-VERSION:3
#EXT-X-MEDIA-SEQUENCE:0
#EXT-X-ALLOW-CACHE:YES
#EXT-X-TARGETDURATION:13
#EXTINF:12.933333,
abc000.ts
#EXT-X-ENDLIST

// mediafilesegmenter
#EXTM3U
#EXT-X-TARGETDURATION:5
#EXT-X-VERSION:3
#EXT-X-MEDIA-SEQUENCE:0
#EXT-X-PLAYLIST-TYPE:VOD
#EXTINF:5.00000,	
#EXT-X-BITRATE:893
TEST0.ts
#EXTINF:5.00000,	
#EXT-X-BITRATE:784
TEST1.ts
#EXTINF:2.67767,	
#EXT-X-BITRATE:722
TEST2.ts
#EXT-X-ENDLIST
```
它们的共同字段是

而在使用

```
http://devimages.apple.com/iphone/samples/bipbop/bipbopall.m3u8
```

它的 m3u8 文件如下

```
#EXTM3U
#EXT-X-STREAM-INF:PROGRAM-ID=1, BANDWIDTH=200000
gear1/prog_index.m3u8
#EXT-X-STREAM-INF:PROGRAM-ID=1, BANDWIDTH=311111
gear2/prog_index.m3u8
#EXT-X-STREAM-INF:PROGRAM-ID=1, BANDWIDTH=484444
gear3/prog_index.m3u8
#EXT-X-STREAM-INF:PROGRAM-ID=1, BANDWIDTH=737777
gear4/prog_index.m3u8
```

总结：

* EXTM3U：所有 HLS 播放列表必须以此标记开头
* EXT-X-ENDLIST：区分是点播还是直播
* 二级目录是 绝对路径还是相对路径
* 二级目录是 ts 文件 还是 m3u8
* 当二级目录是 m3u8 时，需要再次访问获取最终的 ts，而直播就是通过这种不断刷新 m3u8 来实现播放

参考：

* [HTTP Live Streaming (HLS) - 概念](https://www.jianshu.com/p/2ce402a485ca)
* [流媒体开发之--HLS--M3U8解析(1)](https://blog.csdn.net/jwzhangjie/article/details/9743971)
* [流媒体开发之--HLS--M3U8解析(2): HLS草案](https://blog.csdn.net/jwzhangjie/article/details/9744027)


##### TS

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;TS 文件的结构与 HLS 类似，一个个的分段结构。TS 是以 188字节 来分段。
每一段：TS层（Head + Pes层（Head + ES层）），TS层 的内容是通过 PID 值来标识的，主要内容包括：PAT表、PMT表、音频流、视频流。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;因此 TS 文件大致结构图如下：

| TSHead | PAT/PMT | Stuffing Bytes |
| --- | :---: | --- |
| TSHead | Adaptation Field | Pes1 |
| TSHead | \ | Pes2 - Pes(N-1) |
| TSHead | Adaptation Field | PesN |

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;如下参考里面有说明 TSHead、PAT、PMT、Adapt、PES、ES 等字段的内容。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;此处写出本人解码时遇到的几个点

* TSHead 的 payload_unit_start_indicator 对应 PAT & PMT：在前4个字节后会有一个调整字节。所以实际数据应该为去除第一个字节后的数据。即上面数据中红色部分不属于有效数据包。而对于 Adapt & 其他：0x01表示含有PSI或者PES头。所以解析 PAT/PMT 时需要通过此字段来判断是否空一个字节。
* 顺序查找，先是 PID为0 的 PAT，通过其 Programs 里面的 PID 查找 PMT，再又 PMT 里面的 Streams 来查找音视频数据。这也就是 **"解析 ts 流要先找到 PAT 表，只要找到 PAT 就可以找到 PMT，然后就可以找到 音视频流 了"** 的意思
* 在 PAT/PMT 遇到 section_length 这种 length 的解析，它是指后面数据的长度，即从解析出 length 的字节之后开始计算，如果length 的 Index 是 3（index 是从 0 开始的，即 length 是第四个字节），那么整个类型的实际数据长度则为 3 + 1 + length = len。
* PMT 的 stream_type	：流类型，标志是 Video 还是 Audio 还是其他数据，h.264编码 对应 0x1b，aac编码 对应 0x0f，mp3编码 对应 0x03
* adaption 包含 PCR：Program Clock Reference，节目时钟参考，用于恢复出与编码端一致的系统时序时钟 STC（System Time Clock）。

##### ES

(暂未解析)

参考：

* [hls 之 m3u8、ts 流格式详解](https://blog.csdn.net/allnlei/article/details/53005350)
* [SJIE](http://www.lnmcc.net/page21/)
* [TS 数据流分析学习](http://www.cnblogs.com/doscho/p/7678579.html)
* [从 TS 流到 PAT 和 PMT](https://blog.csdn.net/rongdeguoqian/article/details/18214627)
* [ts 文件格式解析](https://blog.csdn.net/occupy8/article/details/43115765)
* [TS 协议解析第一部分（PAT）](https://blog.csdn.net/u013354805/article/details/51578457)
* [TS 协议解析第二部分（PMT）](https://blog.csdn.net/u013354805/article/details/51586086)
* [TS 协议解析第三部分（PES）](https://blog.csdn.net/u013354805/article/details/51591229)
* [TS 协议解析第四部分（adaptation field）](https://blog.csdn.net/u013354805/article/details/51683830)




