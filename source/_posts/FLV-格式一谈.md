---
title: FLV 格式一谈
date: 2018-06-20 13:57:57
tags: 
  - Swift
  - FLV
  - MacOS
categories:
  - 音视频

---
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;本着了解直播行业的初心。那先来看看视频播放的文件格式之一 -- FLV，进行简单访谈。首先通过 ffmpeg 将直播流保存为 flv 文件。

```shell
ffmpeg -i rtmp://server/live/streamName -c copy dump.flv
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;决定自己开发 Mac 上的 FLV 解析工具，源码在此 [QHFlvParserMan](https://github.com/chenqihui/QHFlvParserMan)。原因是在 Mac 上搜索的解析工具只有 App Store 上付费的软件 [FLV Analyzer](https://itunes.apple.com/cn/app/flv-analyzer/id1182196829?mt=12)，这不是打广告，因为我的工具界面是模仿它的。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;工具解析了 FLV 文件的数据类型，各个类型包的大小、时间戳和简单信息。

<!-- more -->

![QHFlvParserMan 工具截图](https://anakinpublicspace-1253727175.cos.ap-chengdu.myqcloud.com/blog/QHFlvParserMan.png?attname=)

工具的实现主要参考：  
* 《[FFmpeg从入门到出家（FLV文件结构解析） - 简书](https://www.jianshu.com/p/d68d6efe8230)》  
* 《[FLV 文件格式 - 小小程序员001 - 博客园](https://www.cnblogs.com/musicfans/archive/2012/11/07/2819291.html) 》  
* 《[FLV科普12 FLV脚本数据解析-Metadata Tag解析 - CSDN博客](https://blog.csdn.net/cabbage2008/article/details/50500021)》

#### 聊回 FLV 格式

>FLV(FLASH VIDEO)，是一种常用的文件封装格式，目前国内外大部分视频分享网站都是采用的这种格式。其标准定义为[《Adobe Flash Video File Format Specification》](https://link.jianshu.com/?t=http%3A%2F%2Fwwwimages.adobe.com%2Fcontent%2Fdam%2Facom%2Fen%2Fdevnet%2Fflv%2Fvideo_file_format_spec_v10_1.pdf)。RTMP协议也是基于FLV视频格式的。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;其实在 金山云 的文章大致已经说明了。这里结合代码说下。简单来说，它由一个 FLV Header 和 多个 FLV body（previousTagSize + tag）组成，而 tag 则分为video、audio或者scripts，也是由一个 Tag Header 和一个 Tag Body 组成。不同的 tag 的 Tag Header 不同。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;以下内容标准都可以参考 [《Adobe Flash Video File Format Specification》](https://link.jianshu.com/?t=http%3A%2F%2Fwwwimages.adobe.com%2Fcontent%2Fdam%2Facom%2Fen%2Fdevnet%2Fflv%2Fvideo_file_format_spec_v10_1.pdf) 

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;源码 Demo [QHFlvParserMan](https://github.com/chenqihui/QHFlvParserMan) 是采用 Swift 实现的，内容可结合此工程进行了解：

#### FLV Header

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;版本为 1 的FLV，其 Header 共 9 个字节，字段如下：

| Signature | Version | TypeFlags | DataOffset |   
| --------- | ------- | --------- | ---------- |  
| 3 Bytes   | 1 Byte | 1 Byte 	| 4 Bytes |  
| 文件标识   | 版本号	 | 文件包含的内容（音频、视频或者音视频 | 长度，版本为 1 时，该值为 9 |  

| | Type | Binary Type | Comment |   
| --------- | :-------: | --------- | ---------- |
| **TypeFlags** 	| 1 | 0000 0001 | only video |  
|  					| 4 | 0000 0100 | only audio |   
|  					| 5 | 0000 0101 | audio & video |  

```swift
QHFlvParser+Tag.swift
```

#### FLV Body

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;FLV File Body 是由一系列的PreviousTagSize + Tag 组成，其中 PreviousTagSize 的长度为 4 个字节，用来表示前一个 Tag 的长度；Tag 里面的数据可能是 video、audio 或者 scripts 等。

| PreviousTag Size 0 | Tag 1 | PreviousTag Size 1 | ... | PreviousTag Size N - 1 | Tag N |  
| --------- | ---------- | ---- | ---- | ---- | ---- |  
| 4 Bytes   		 | N Bytes | 4 Bytes 	|  | 4 Bytes | M Bytes |  
| 前一个 Tag 的长度 | 数据 | 前一个 Tag 的长度 | | | | |  

##### PreviousTagSize

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;共 4 个字节

```swift
QHFlvParser+Tag.swift
```

##### Tag

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Tag里面的数据可能是 video、audio 或者 scripts 等，它们都由 header + body 组成。

##### 1、header

| Signature | Filter | TagType | DataSize | Timestamp | Timestamp Extended | StreamID |   
| --------- | ------- | ---- | ---- | ---- | :----: | ---- |   
| 2 bits | 1 bit | 5 bits 	| 24 bits | 24 bits | 8 bits | 24 bits |  
|    | 加扰标识	 | 数据类型 | StreamID 之后的数据长度 | 时间戳 | 扩展时间戳 | ID（总是0） |  

| | Type | Binary Type | Comment |   
| --------- | :-------: | --------- | ---------- |
| **TagType** 	| 8 	| 0 1000 | audio |  
|  					| 9		| 0 1001 | video |   
|  					| 18 	| 1 0010 | cripts | 

>PTS = Timestamp | TimestampExtended<<24  
>PTS（Presentation Time Stamp）：即显示时间戳，这个时间戳用来告诉播放器该在什么时候显示这一帧的数据。

[理解音视频 PTS 和 DTS | SamirChen](http://www.samirchen.com/about-pts-dts/)

```swift
QHFlvParser+Tag.swift
```

##### 2、body

| Script/Audio/Video TagHeader | Script/Audio/Video TagBody |   
| --------- | ------- |  
| 头部信息 | 内容数据 |  

##### 2.1、script data

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;如果 TAG 包中的 TagType 等于18，表示该 Tag 中包含的数据类型为 SCRIPT。SCRIPTDATA 结构十分复杂，定义了很多格式类型，每个类型对应一种结构。

scriptdata = Type + body

```swift
QHFlvParser+Script.swift
```

| Field | Type | Comment |  
| --- | --- | --- |  
| Type | UI8 | UI8	Type of the ScriptDataValue.<br>The following types are defined:<br>0 = Number<br>1 = Boolean<br>2 = String<br>3 = Object<br>4 = MovieClip (reserved, not supported)<br>5 = Null<br>6 = Undefined<br>7 = Reference<br>8 = ECMA array<br>9 = Object end marker<br>10 = Strict array<br>11 = Date<br>12 = Long string |  
| ScriptDataValue | IF Type == 0<br>DOUBLE<br>IF Type == 1<br>UI8<br>IF Type == 2<br>SCRIPTDATASTRING<br>IF Type == 3<br>SCRIPTDATAOBJECT<br>IF Type == 7<br>UI16<br>IF Type == 8<br>SCRIPTDATAECMAARRAY<br>IF Type == 10<br>SCRIPTDATASTRICTARRAY<br>IF Type == 11<br>SCRIPTDATADATE<br>IF Type == 12<br>SCRIPTDATALONGSTRING | Script data value.<br>The Boolean value is (ScriptDataValue ≠ 0). |  

##### 2.2、onMetaData

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;它是 SCRIPTDATA 中一个非常重要的信息。它通常是 FLV 文件中的第一个 Tag，用来表示当前文件的一些基本信息: 比如视音频的编码类型id、视频的宽和高、文件大小、视频长度、创建日期等。

| Property Name | Type | Comment |  
| --- | --- | --- | --- |  
| audiocodecid | Number	 | Audio codec ID used in the file (see E.4.2.1 for available SoundFormat values) |  
| audiodatarate | Number | Audio bit rate in kilobits per second |  
| audiodelay | Number | Delay introduced by the audio codec in seconds |  
| audiosamplerate | Number | Frequency at which the audio stream is replayed |  
| audiosamplesize | Number | Resolution of a single audio sample |  
| canSeekToEnd | Boolean | Indicating the last video frame is a key frame |  
| creationdate | String | Creation date and time |  
| duration | Number | Total duration of the file in seconds |  
| filesize | Number | Total size of the file in bytes |  
| framerate | Number | Number of frames per second |  
| height | Number | Height of the video in pixels |  
| stereo | Boolean | Indicating stereo audio |  
| videocodecid | Number	 | Video codec ID used in the file (see E.4.3.1 for available CodecID values) |  
| videodatarate | Number | Video bit rate in kilobits per second |  
| width | Number | Width of the video in pixels |  |  

##### 2.2、audio

| SoundFormat | SoundRate | SoundSize | SoundType | AACPacketType | AudioTagBody |   
| --------- | ------- | ---- | ---- | ---- | ---- |    
| 4 bits | 2 bits | 1 bit 	| 1 bit | 8 bits |  | 
| 编码格式   | 采样率	 | 采样点位宽 | 立体声标识 | 表示接下来 AUDIODATA 的内容  | AudioSpecificConfig / data | |

| | Type | Comment |   
| --------- | :-------: | ---------- |
| **SoundFormat** 	| 0 	| Linear PCM, platform endia |  
|  						| 1 	| ADPCM |   
|  						| 2 	| MP3 |
|  						| 3		| Linear PCM, little endian |   
|  						| 4 	| Nellymoser 16 kHz mono |   
|  						| 5 	| Nellymoser 8 kHz mono |   
|  						| 6 	| Nellymoser |   
|  						| 7 	| G.711 A-law logarithmic PCM |   
|  						| 8 	| G.711 mu-law logarithmic PCM |   
|  						| 9 	| reserved |   
|  						| 10 	| AAC |   
|  						| 11 	| Speex |   
|  						| 14 	| MP3 8 kHz |   
|  						| 15 	| Device-specific sound | 

| | Type | Comment |   
| --------- | :-------: | ---------- |
| **SoundRate** 		| 0 	| 5.5 kHz |  
|  						| 1 	| 11 kHz |  
|  						| 2 	| 22 kHz |  
|  						| 3 	| 44 kHz |  

| | Type | Comment |   
| --------- | :-------: | ---------- |
| **SoundSize** 		| 0 	| 8-bit samples |  
|  						| 1 	| 16-bit samples |  

| | Type | Comment |   
| --------- | :-------: | ---------- |
| **SoundType** 		| 0 	| Mono sound |  
|  						| 1 	| Stereo sound |  

接下来主要介绍 AAC：  

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**AACPacketType：需要说明的是，通常情况下 AudioTagHeader 之后跟着的就是 AUDIODATA 数据了，但有个特例，如果音频编码格式为 AAC，AudioTagHeader 中会多出1个字节的数据 AACPacketType，这个字段来表示 AACAUDIODATA 的类型：**

| | Type | Comment |   
| --------- | :-------: | ---------- |
| **AACPacketType** | 0 	| AAC sequence header (AudioSpecificConfig) |  
|  						| 1 	| AAC raw |  

>为什么 AudioTagHeader 中定义了音频的相关参数，我们还需要传递AudioSpecificConfig 呢？
>
>因为当 SoundFormat 为 AAC 时，SoundType 须设置为1（立体声），SoundRate 须设置为3（44KHZ），但这并不意味着 FLV 文件中 AAC 编码的音频必须是 44KHZ 的立体声。播放器在播放 AAC 音频时，应忽略 AudioTagHeader 中的参数，并根据AudioSpecificConfig 来配置正确的解码参数。  

&

>在 FLV 的文件中，一般情况下 AAC sequence header 这种包只出现1次，而且是第一个audio tag，为什么要提到这种 tag，因为当时在做 FLV demux 的时候，如果是 AAC 的音频，需要在每帧 AAC ES流 前边添加 7 个字节 ADST 头, ADST 在音频的格式中会详细解读，这是解码器通用的格式，就是 AAC 的纯 ES流 要打包成 ADST 格式的 AAC 文件，解码器才能正常播放.就是在打包 ADST 的时候，需要 samplingFrequencyIndex 这个信息，samplingFrequencyIndex 最准确的信息是在 AudioSpecificConfig 中，所以就对 AudioSpecificConfig 进行解析并得到了 samplingFrequencyIndex。

```swift
QHFlvParser+Audio.swift
```

##### 2.2.1、AudioSpecificConfig

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;针对 AudioSpecificConfig，工具代码还没实现。进一步了解的话，需要查阅 ISO 标准或者参考 ffmpeg 实现的解析逻辑。

>AAC sequence header 中存放的是 AudioSpecificConfig，该结构包含了更加详细的音频信息，《ISO-14496-3 Audio》中的 1.6.2.1 章节对此作了详细定义。
>
>通常情况下，AAC sequence header 这种 Tag 在FLV文件中只出现1次，并且是第一个Audio Tag ，它存放了解码AAC音频所需要的详细信息。
>
>有关 AudioSpecificConfig 结构的代码解析，可以参考 ffmpeg/libavcodec/mpeg4audio.c 中的 [avpriv_mpeg4audio_get_config](https://www.jianshu.com/writer#L155) 方法。
>
>为什么 AudioTagHeader 中定义了音频的相关参数，我们还需要传递 AudioSpecificConfig 呢？
>
>因为当 SoundFormat 为 AAC 时，SoundType 须设置为1（立体声），SoundRate 须设置为3（44KHZ），但这并不意味着 FLV 文件中 AAC 编码的音频必须是 44KHZ 的立体声。播放器在播放 AAC 音频时，应忽略 AudioTagHeader 中的参数，并根据 AudioSpecificConfig 来配置正确的解码参数。


##### 2.3、video

| FrameType | CodecID | AVCPacketType | CompositionTime | VideoTagBody |   
| --------- | ------- | ---- | ---- | ---- |    
| 4 bits | 4 bits | 8 bits 	| 24 bits |  | 
| 关键帧标识   | 编码格式	 | 表示 VIDEODATA 的内容 | 相对时间戳 | AVCDecoderConfigurationRecord / data | |

| | Type | Comment |   
| --------- | :-------: | ---------- |  
| **FrameType**		| 1 	| key frame (for AVC, a seekable frame) |   
|  						| 2 	| inter frame (for AVC, a non-seekable frame) |
|  						| 3		| disposable inter frame (H.263 only) |   
|  						| 4 	| generated key frame (reserved for server use only) |   
|  						| 5 	| video info/command frame |    

| | Type | Comment |   
| --------- | :-------: | ---------- |  
| **CodecID**			| 2 	| Sorenson H.263 |   
|  						| 3 	| Screen video |
|  						| 4		| On2 VP6 |   
|  						| 5 	| On2 VP6 with alpha channel |   
|  						| 6 	| Screen video version 2 |    
|  						| 7 	| AVC |  

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**VideoTagHeader 之后跟着的就是 VIDEODATA 数据了，但是和 AAC 音频一样，它也存在一个特例，就是当视频编码格式为 H.264 的时候，VideoTagHeader 会多出4个字节的信息，AVCPacketType 和 CompositionTime 。** 

when CodecID == 7:

| | Type | Comment |   
| --------- | :-------: | ---------- |  
| **AVCPacketType**	| 0 	| AVC sequence header |   
|  						| 1 	| AVC NALU |
|  						| 2		| AVC end of sequence (lower level NALU sequence ender is not required or supported) |  

| | condition | Comment |   
| --------- | :-------: | ---------- |  
| **CompositionTime**	| AVCPacketType == 1 | Composition time offset |
|  											| AVCPacketType != 1 | 0 |   

```swift
QHFlvParser+Video.swift
```

##### 2.3.1、AVCDecoderConfigurationRecord

>AVCDecoderConfigurationRecord 包含着是 H.264 解码相关比较重要的 sps 和 pps 信息，再给 AVC 解码器送数据流之前一定要把 sps 和 pps 信息送出，否则的话解码器不能正常解码。而且在解码器 stop 之后再次 start 之前，如 seek、快进快退状态切换等，都需要重新送一遍 sps 和 pps 的信息。 AVCDecoderConfigurationRecord 在 FLV 文件中一般情况也是出现1次，也就是第一个 video tag。

>有关 AVCDecoderConfigurationRecord 结构的代码解析，可以参考中的[ff_isom_write_avcc](https://www.jianshu.com/writer#L107) 方法。

2.2.2、CompositionTime (相对时间戳)

>相对时间戳的概念需要和 PTS、DTS 一起理解：
>
>DTS : Decode Time Stamp，解码时间戳，用于告知解码器该视频帧的解码时间；
>PTS : Presentation Time Stamp，显示时间戳，用于告知播放器该视频帧的显示时间；  
>CTS : Composition Time Stamp，相对时间戳，用来表示 PTS 与 DTS 的差值。
>
>如果视频里各帧的编码是按输入顺序依次进行的，则解码和显示时间相同，应该是一致的。但在编码后的视频类型中，如果存在 B 帧，输入顺序和编码顺序并不一致，所以才需要 PTS 和 DTS 这两种时间戳。视频帧的解码一定是发生在显示前，所以视频帧的 PTS，一定是大于等于 DTS 的，因此 CTS = PTS - DTS。
>
>FLV Video Tag 中的 TimeStamp，不是 PTS，而是 DTS，视频帧的 PTS 需要我们通过 DTS + CTS 计算得到。
>
>为什么 Audio Tag 不需要 CompositionTime 呢？
>
>因为 Audio 的编码顺序和输入顺序一致，即 PTS = DTS，所以它没有 CompositionTime 的概念。


#### 扩展阅读

[《ISO-14496-3-2016》.pdf文档全文免费阅读、在线看](https://max.book118.com/html/2015/1019/27550235.shtm)
[bit ( 比特 )和 Byte（字节）的关系 以及 网速怎么算 - 未曾见海 - 博客园](https://www.cnblogs.com/afei-qwerty/p/6667110.html)  
[基于FLV视频的RTMP和HTTP区别 - CSDN博客](https://blog.csdn.net/frankiewang008/article/details/44834925)  
[理解RTMP、HttpFlv和HLS的正确姿势 - 简书](https://www.jianshu.com/p/32417d8ee5b6)  
[直播未来属于RTMP还是HTTP？ - Tinywan - 博客园](https://www.cnblogs.com/tinywan/p/6122065.html)

