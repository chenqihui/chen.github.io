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
本着了解直播行业的初心。那先来看看视频播放的文件格式之一 -- FLV，进行简单访谈。首先通过 ffmpeg 将直播流保存为 flv 文件

```
ffmpeg -i rtmp://server/live/streamName -c copy dump.flv
```
决定自己开发 Mac 上的 FLV 解析工具，源码在此 [QHFlvParserMan](https://github.com/chenqihui/QHFlvParserMan)。原因是在 Mac 上搜索的解析工具只有 App Store 上付费的软件 [FLV Analyzer](https://itunes.apple.com/cn/app/flv-analyzer/id1182196829?mt=12)，这不是打广告，因为我的工具界面是模仿它的。

工具解析了 FLV 文件的数据类型，各个类型包的大小、时间戳和简单信息。

![QHFlvParserMan 工具截图](http://pacfu36li.bkt.clouddn.com/QHFlvParserMan.png?attname=)

工具的实现主要参考：《[FFmpeg从入门到出家（FLV文件结构解析） - 简书](https://www.jianshu.com/p/d68d6efe8230)》 

#### 聊回 FLV 格式

>FLV(FLASH VIDEO)，是一种常用的文件封装格式，目前国内外大部分视频分享网站都是采用的这种格式。其标准定义为[《Adobe Flash Video File Format Specification》](https://link.jianshu.com/?t=http%3A%2F%2Fwwwimages.adobe.com%2Fcontent%2Fdam%2Facom%2Fen%2Fdevnet%2Fflv%2Fvideo_file_format_spec_v10_1.pdf)。RTMP协议也是基于FLV视频格式的。

其实在 金山云 的文章大致已经说明了。这里结合代码说下。简单来说，它由一个 FLV Header 和 多个 FLV body（previousTagSize + tag）组成，而 tag 则分为video、audio或者scripts，也是由一个 Tag Header 和一个 Tag Body 组成。不同的 tag 的 Tag Header 不同。

以下内容标准都可以参考 [《Adobe Flash Video File Format Specification》](https://link.jianshu.com/?t=http%3A%2F%2Fwwwimages.adobe.com%2Fcontent%2Fdam%2Facom%2Fen%2Fdevnet%2Fflv%2Fvideo_file_format_spec_v10_1.pdf) 

#### FLV Header

它共 9 个字节，包含 FLV 文件标识，版本号，文件包含的内容（音频、视频或者音视频）和长度。

| Signature | Version | TypeFlags | DataOffset |   
| --------- | ------- | --------- | ---------- |  
| 3 Bytes   | 1 Byte | 1 Byte 	| 4 Bytes |  
| 文件标识   | 版本	 | 文件包含的内容 | 长度 |  

```ObjC
if isFlvFile() {
    print("是 flv 文件")
    print("版本：\(version())")
    print("类型：\(type())")
    print("头部长度：\(headerLength())")
}
else {
    print("不是 flv 文件")
}
```

#### FLV Body

FLV File Body是由一系列的PreviousTagSize + Tag组成，其中PreviousTagSize的长度为4个字节，用来表示前一个Tag的长度；Tag里面的数据可能是video、audio或者scripts。

| PreviousTagSize0 | Tag1 | PreviousTagSize1 | ... | TagN | PreviousTagSizeN |  
| --------- | ---------- | ---- | ---- | ---- | ---- |  
| 4 Bytes   | N Byte | 4 Byte 	|  | N Byte | 4 Byte |  
| 前一个Tag的长度   | 数据	 | | | | |  

##### PreviousTagSize

共 4 个字节

```ObjC
private func previousTagSize() -> uint {
    if fileData.count > flvOffset + previousTagSizeBytes {
        let v1 = uint(fileData[Int(flvOffset + 1)])
        let v2 = uint(fileData[Int(flvOffset + 2)])
        let v3 = uint(fileData[Int(flvOffset + 3)])
        let v4 = uint(fileData[Int(flvOffset + 4)])
        let l1 = v1 * 1<<24
        let l2 = v2 * 1<<16
        let size = l1 + l2 + v3 * 1<<8 + v4
        
        return size
    }
    return 0
}
```


##### Tag

Tag里面的数据可能是video、audio或者scripts

1、header

| Signature | Filter | TagType | DataSize | Timestamp | TimestampExtended | StreamID |   
| --------- | ------- | ---- | ---- | ---- | ---- | ---- |   
| 2 bits | 1 bit | 5 bits 	| 24 bits | 24 bits | 8 bits | 24 bits |  
|    | 加扰标识	 | 数据类型 | 长度 | 时间戳 | 扩展时间戳 | ID |  

```
//2.2、Tag里面的数据可能是video、audio或者scripts
var tag = QHFlvTag()
if fileData.count > flvOffset + tagSizeBytesExceptHeaderAndBody {
    let v1 = uint(fileData[Int(flvOffset + 1)])
    //2.2.1、0x08, 二进制为0000 1000，第3位为0, 表示为非加扰文件;
    if v1 & 0b00100000 == 0b00100000 {
        //加扰文件
        tag.signature = true
    }
    else {
        //非加扰文件
        tag.signature = false
    }
    //2.2.2、低5位01000为8，说明这个Tag包含的数据类型为Audio；
    let v1_2 = v1 & 0b00011111
    if v1_2 == 8 {
        tag.tagType = .audio
    }
    else if v1_2 == 9 {
        tag.tagType = .video
    }
    else if v1_2 == 18 {
        tag.tagType = .script
    }
    //2.2.4、Tag的内容长度为，与该tag后面的previousTagSize() - 11相同；
    let v2 = uint(fileData[Int(flvOffset + 2)])
    let v3 = uint(fileData[Int(flvOffset + 3)])
    let v4 = uint(fileData[Int(flvOffset + 4)])
    
    tag.dataSize = v2 * 1<<16 + v3 * 1<<8 + v4
    
    //2.2.5、当前Audio数据的时间戳；
    let v5 = uint(fileData[Int(flvOffset + 5)])
    let v6 = uint(fileData[Int(flvOffset + 6)])
    let v7 = uint(fileData[Int(flvOffset + 7)])
    
    tag.timestamp = v7 * 1<<16 + v6 * 1<<8 + v5
    
    //2.2.6、扩展时间戳，如果扩展时间戳不为0，那么该Tag的时间戳应为：Timestamp | TimestampExtended<<24；
    let v8 = uint(fileData[Int(flvOffset + 8)])
    
    tag.timestampExtended = v8
    
    //2.2.7、StreamID
    let v9 = uint(fileData[Int(flvOffset + 9)])
    let v10 = uint(fileData[Int(flvOffset + 10)])
    let v11 = uint(fileData[Int(flvOffset + 11)])
    
    tag.streamID = v11 * 1<<16 + v10 * 1<<8 + v9
}
```

2、body

| Audio/Video TagHeader | Audio/Video TagBody |   
| --------- | ------- |  

2.1、scripts

***[TODO] 后期需要再开发解析 scripts***。

如果TAG包中的TagType等于18，表示该Tag中包含的数据类型为SCRIPT。

SCRIPTDATA 结构十分复杂，定义了很多格式类型，每个类型对应一种结构，详细可参考E.4.4 Data Tags

onMetaData是SCRIPTDATA中一个非常重要的信息，其结构定义可参考 [E.5 onMetaData](https://link.jianshu.com/?t=http%3A%2F%2Fwwwimages.adobe.com%2Fcontent%2Fdam%2Facom%2Fen%2Fdevnet%2Fflv%2Fvideo_file_format_spec_v10_1.pdf)。它通常是FLV文件中的第一个Tag，用来表示当前文件的一些基本信息: 比如视音频的编码类型id、视频的宽和高、文件大小、视频长度、创建日期等。


2.2、audio

| SoundFormat | SoundRate | SoundSize | SoundType | AACPacketType | AudioTagBody |   
| --------- | ------- | ---- | ---- | ---- | ---- |    
| 4 bits | 2 bits | 1 bit 	| 1 bit | 8 bits |  | 
| 编码格式   | 采样率	 | 采样点位宽 | 立体声标识 | AudioSpecificConfig |  |

```
//3、audio data
private func audioParser(audioData: Data) -> QHAudioTag {
    var audioTag = QHAudioTag()
    let v1 = uint(audioData[audioData.startIndex])
    //3.1、高4位为1010，转十进制为10，表示Audio的编码格式为AAC；
    audioTag.soundFormat = uint(v1>>4)//10为AAC
    //3.2、第3、2位为11，转十进制为3，表示该音频的采样率为44KHZ；
    let v1_2 = v1 & 0b00001100
    if v1_2>>2 == 3 {
        audioTag.soundRate = "44KHZ"
    }
    //3.3、第1位为1，表示该音频采样点位宽为16bits；
    let v1_3 = v1 & 0b00000010
    if v1_3>>1 == 1 {
        audioTag.soundSize = "16bits"
    }
    //3.4、第0位为1，表示该音频为立体声。
    let v1_4 = v1 & 0b00000001
    if v1_4 == 1 {
        audioTag.soundType = "立体声"
    }
    //3.5、[3.3.1 AudioSpecificConfig]
    if audioTag.soundFormat == 10 {
        let v1 = uint(audioData[audioData.startIndex + 1])
        audioTag.soundRate = "44KHZ"
        audioTag.soundSize = "16bits"
        audioTag.soundType = "立体声"
        
        //3.6、十进制为0，并且Audio的编码格式为AAC，说明AACAUDIODATA中存放的是AAC sequence header；
        audioTag.accPackType = v1
        //3.7、AUDIODATA数据，即AAC sequence header。
        audioTag.audioBody = audioData[audioData.startIndex + 2..<audioData.endIndex]
    }
    else {
        audioTag.audioBody = audioData[audioData.startIndex + 1..<audioData.endIndex]
    }
    return audioTag
}
```


2.3、video

| FrameType | CodecID | AVCPacketType | CompositionTime | VideoTagBody |   
| --------- | ------- | ---- | ---- | ---- |    
| 4 bits | 4 bits | 8 bits 	| 24 bits |  | 
| 关键帧标识   | 编码格式	 | 相对时间戳 | AVCDecoderConfigurationRecord |  |

```
//4、video data
private func videoParser(videoData: Data) -> QHVideoTag {
    var videoTag = QHVideoTag()
    let v1 = uint(videoData[videoData.startIndex])
    //4.1、高4位为0001，转十进制为1，表示当前帧为关键帧；
    let v1_1 = v1 & 0b11110000
    if v1_1>>4 == 1 {
        videoTag.frameType = 1
    }
    //4.2、低4位为0111，转十进制为7，说明当前视频的编码格式为AVC
    let v1_2 = v1 & 0b00001111
    videoTag.codecID = v1_2//7为AVC
    //4.3、十进制为0，并且Video的编码格式为AVC，说明VideoTagBody中存放的是AVC sequence header
    let v2 = uint(videoData[videoData.startIndex + 1])
    
    //4.4、VIDEODATA的内容
    videoTag.avcPackType = v2
    
    let v3 = uint(videoData[videoData.startIndex + 2])
    let v4 = uint(videoData[videoData.startIndex + 3])
    let v5 = uint(videoData[videoData.startIndex + 4])
    
    //4.5、CompositonTime相对时间戳，如果AVCPacketType=0x01，为相对时间戳，其它均为0；
    videoTag.compositionTime = v3 * 1<<16 + v4 * 1<<8 + v5
    
    //4.6、VIDEODATA数据，即AVC sequence header
    videoTag.videoBody = videoData[videoData.startIndex + 5..<videoData.endIndex]
    
    return videoTag
}
```

#### 扩展阅读

[bit ( 比特 )和 Byte（字节）的关系 以及 网速怎么算 - 未曾见海 - 博客园](https://www.cnblogs.com/afei-qwerty/p/6667110.html)  
[基于FLV视频的RTMP和HTTP区别 - CSDN博客](https://blog.csdn.net/frankiewang008/article/details/44834925)  
[理解RTMP、HttpFlv和HLS的正确姿势 - 简书](https://www.jianshu.com/p/32417d8ee5b6)  
[直播未来属于RTMP还是HTTP？ - Tinywan - 博客园](https://www.cnblogs.com/tinywan/p/6122065.html)

