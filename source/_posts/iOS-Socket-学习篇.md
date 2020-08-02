---
title: iOS Socket 学习篇
date: 2019-03-04 01:23:14
tags:
  - Socket
  - TCP / UDP / IP
  - WebSocket
categories:
  - iOS
  
---
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;谈谈 Socket 的大纲

* 概念
* 基本操作 Socket、CFSocket、GCDCocoaAysnSocket
* 第三方封装 SocketRocket、Pemolo、SocketIO
* 简单功能：测试 & 域名访问 SimplePing
* 服务端（Mac）
* 数据格式：使用 二进制 或者 protocolbuffer 
* 模拟 HTTP 和 HTTPS（顺序学习 HTTP 协议头）
* 抓包 基本协议、了解下 TCP、UDP
* WebSocket 协议（WebRTC）

<!-- more -->

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Socket 在很多场景都使用上，如 IM、直播间公屏、推送、推流等等。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;本博文所使用的 [chenqihui/QHSocketDemo](https://github.com/chenqihui/QHSocketDemo) 项目例子。


#### 概念

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;伯克利套接字（英语：Internet Berkeley sockets） ，又称为BSD 套接字(BSD sockets)是一种应用程序接口（API），用于网络套接字（ socket）与Unix域套接字，包括了一个用C语言写成的应用程序开发库，主要用于实现进程间通讯，在计算机网络通讯方面被广泛使用。

* [Berkeley套接字](https://zh.wikipedia.org/wiki/Berkeley%E5%A5%97%E6%8E%A5%E5%AD%97)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;作为介绍 Socket，墙裂推荐下面的文章，服务端的 node.js 也是采用下文作者提供的（只是增加了 SocketIO）。

* [iOS即时通讯，从入门到“放弃”？](https://www.jianshu.com/p/2dbb360886a8)
* [深入浅出：iOS 的 TCP/IP 协议族剖析 && Socket - IOS](http://ios.jobbole.com/84039/)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Socket 是在 TCP、UDP等协议基础上实现进程间通讯的封装。

![](https://upload-images.jianshu.io/upload_images/2702646-5eaf55a2a5469d4e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/640/format/webp.p)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;而选择哪种协议可以测试在目前移动网络环境下，哪种更加稳定和功耗更小：

* [移动端IM/推送系统的协议选型：UDP还是TCP](http://www.52im.net/thread-33-1-1.html)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;其他

* [INADDR\_ANY](https://baike.baidu.com/item/INADDR_ANY/1493998)


#### 基本操作

##### C Socket

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;TCP 服务器端：  
socket() -> bind() -> listen() -> accept() -阻塞直到有客户端连接-> read() -> write() -> read() -> close()  
创建套接字 -> 绑定本机端口 -> 侦听端口 -> 建立连接 - 等待客户端连接 -> 数据传输（请求、回应、请求数据 -> 结束连接  

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;TCP 客户端：  
socket() -> connect -> write() -> read() -> close()  
创建套接字 -> 建立连接 -> 数据传输（回应、请求数据） -> 结束连接  

![](https://upload-images.jianshu.io/upload_images/1156719-f11be16e57524586.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/489/format/webp.p)

创建 Socket，SOCKET_STREAM 是实现 TCP，SOCKET_DGRAM 则是实现 UDP

~~~
/*
一 三种类型的套接字：
1.流式套接字（SOCKET_STREAM)
    提供面向连接的可靠的数据传输服务。数据被看作是字节流，无长度限制。例如FTP协议就采用这种。
2.数据报式套接字（SOCKET_DGRAM）
    提供无连接的数据传输服务，不保证可靠性。
3.原始式套接字（SOCKET_RAW）
    该接口允许对较低层次协议，如IP，ICMP直接访问。
*/
int socket(int domain, int type, int protocol);
~~~

```
// 1、使用子线程进行 while 访问是否有数据
while ((recvLen = recv(self.socketClient, buf, sizeof(buf), 0))) {...}

// 2、在关闭 Socket 的时候，有可能 Host & Port 会进入等待一段时间，而立即关闭可设置
BOOL bDontLinger = FALSE;
setsockopt(socket, SOL_SOCKET, SO_LINGER, (const char*)&bDontLinger, sizeof(BOOL));
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**问题：**

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;服务端与客户端连接成功之后，服务端关闭，而客户端未关闭（或者在其之后关闭），会造成使用的端口暂时无法继续使用

* [socket 通信关于bind那点事1 - king16304的博客](https://blog.csdn.net/king16304/article/details/52278322)
* [linux网络编程之socket（十）：shutdown 与 close 函数 的区别 - Meditation](https://blog.csdn.net/jnu_simba/article/details/9068059)
* [recv()](https://baike.baidu.com/item/recv%28%29/10082153?fr=aladdin)
* [(socket)如何解除绑定bind - cym64039的专栏](https://blog.csdn.net/cym64039/article/details/18269983)
* [tcp 服务端如何判断客户端断开连接 - youxin](https://www.cnblogs.com/youxin/p/4056041.html)


##### CFSocket

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;它是 iOS 基于 BSD socket 实现的一套通讯 Socket。

~~~
// 1、cfsocketref 怎么断开？
CFSocketInvalidate(s);  //closes the socket, unless you set the option to not close on invalidation
CFRelease(s);  //balance the create

// 2、while read 时候需要判断读取状态，需要对 Socket 的错误 & 终止情况进行处理
~~~

* [CFSocket | Apple Developer Documentation](https://developer.apple.com/documentation/corefoundation/cfsocket-rg7)
* [iOS 开发之CFSocket](https://www.jianshu.com/p/9353105a9129)
* [iOS刨根问底-深入理解RunLoop - KenshinCui](https://www.cnblogs.com/kenshincui/p/6823841.html)

##### CocoaAsyncSocket

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;它是谷歌的开发者，基于BSD-Socket写的一个IM框架，它给Mac和iOS提供了易于使用的、强大的异步套接字库，向上封装出简单易用OC接口。

* [robbiehanson/CocoaAsyncSocket](https://github.com/robbiehanson/CocoaAsyncSocket)
* [iOS即时通讯进阶 - CocoaAsyncSocket源码解析(Read篇)](https://www.jianshu.com/p/fdd3d429bdb3)
* [【iOS开发】之CocoaAsyncSocket使用](http://www.cocoachina.com/ios/20170615/19529.html)

##### 总结

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;而以上都是基于最底层的 Socket 实现，虽然在使用的 API 上略微有点不同，可是如果直接使用他们需要处理以下问题

* 心跳
* 粘包 & 断包

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;它们的处理在上面的参考文章都有所提及。而粘包 & 断包 的处理可最常使用为以下三种

* 协议头，如 JSON 这类格式，使用长度等参数记录
* 分隔符
* 固定长度


#### 第三方封装

##### SocketRocket

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;它是 Objective-C 版的 WebSocket，使用的是 WebSocket 协议，此协议会在后面说明

* [facebook/SocketRocket: A conforming Objective-C WebSocket client library.](https://github.com/facebook/SocketRocket)

##### SocketIO

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;有很多人经常讲 Socket.io 与 websocket 搞混，实际上他们并不完全等同。它一个完全由 JavaScript 实现、基于Node.js、支持 WebSocket 协议用于实时通信、跨平台的开源框架，它包括了客户端的JavaScript和服务器端的Node.js。也就是说 Socket.io 将 Websocket 和轮询（Polling）机制以及其它的实时通信方式封装成了通用的接口，并且在服务端实现了这些实时通信机制。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Socket.io 中 主要使用了 websocket，将轮询作为其辅助选项，提供的是相同的接口。其与 node.js 一样，也是事件驱动的。

* [Socket.IO — Docs | Socket.IO](https://socket.io/docs/#Installing)
* [socketio/socket.io: Realtime application framework (Node.JS server)](https://github.com/socketio/socket.io)
* [nuclearace/Socket.IO-Client-Swift: socket.io-client for Swift](https://github.com/nuclearace/Socket.IO-Client-Swift)

##### Pomelo

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;网易的 pomelo 协议就是基于 SocketRocket\SocketIO 进行封装。它的分布式体现在了服务端。

* [NetEase](https://github.com/NetEase)
* [pomelo架构概览 · NetEase/pomelo Wiki](https://github.com/NetEase/pomelo/wiki/pomelo%E6%9E%B6%E6%9E%84%E6%A6%82%E8%A7%88)

1、SocketIO-Pomelo 

* [NetEase/pomelo-ioschat: A chat demo for pomelo iOS client](https://github.com/NetEase/pomelo-ioschat)

2、SRWebsocket-Pomelo

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;将 Pomelo 只使用 SRWebSocket

* [Websocket-Pomelo/PomeloClient.m at GeforceLee/Websocket-Pomelo](https://github.com/GeforceLee/Websocket-Pomelo/blob/480ec196bec4bba4df48d18612f1edbdf5d375f9/Client/Client/Pomelo/lib/PomeloClient.m)

##### 总结

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;可以发现上述的第三方封装清一色的使用 WebSocket 协议。可以提前了解下协议。

* [WebSocket、Socket、TCP、HTTP区别 - 江召伟](https://www.cnblogs.com/jiangzhaowei/p/8781635.html)


#### SimplePing

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;使用它可以进行域名访问的简单测试，在 [《iOS 网速监控 学习篇》4.3. SimplePing](http://chenqihui.github.io/2018/09/05/iOS-%E7%BD%91%E9%80%9F%E7%9B%91%E6%8E%A7-%E5%AD%A6%E4%B9%A0%E7%AF%87/#SimplePing) 也有提及。

* [SimplePing](https://developer.apple.com/library/archive/samplecode/SimplePing/Introduction/Intro.html#//apple_ref/doc/uid/DTS10000716-Intro-DontLinkElementID_2)
* [iOS ping - SimplePing 源码解读](https://www.jianshu.com/p/106e35daff87)


#### 服务端（Mac）
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;node.js 服务器：[QHSocketDemo/服务端](https://github.com/chenqihui/QHSocketDemo/tree/master/%E6%9C%8D%E5%8A%A1%E7%AB%AF\(node.js\))

```shell
// node.js 
node server.js

// 或者在 mac命令行终端 输入
nc -lk [port]
```


#### 传输格式

##### 二进制

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Socket 传输都是 NSData.bytes。这里的二进制指的是协议，一般在接口传入 NSString/NSDate，最常采用类似是 json 或者 xml 等格式。但会增加多余的字段，而如果采取约定好的字段，如下

length type value  
4	4	length  

电话	11 String 13812345678  
姓名	3 String 某某某  
性别	1 int 1/男  
婚姻	1 int 0/未婚  

爱好	4 String 游泳、跑步、篮球、唱歌  

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;通过使用 length + value 记录，即写入或者读取都先计算长度，在得到value，当然这里省略了 key，也就是取值顺序是固定的

```objc
- (void)readChar:(NSData *)data {
    int nOffset = 0;
    NSString *t = [self readStringWithData:data output:&nOffset];
    NSString *n = [self readStringWithData:data output:&nOffset];
    NSInteger s = [self readIntWithData:data output:&nOffset];
    NSInteger h = [self readIntWithData:data output:&nOffset];
    NSLog(@"%@,%@,%li,%li", t, n, (long)s, (long)h);
}
```

##### ProtocolBuffer

* [iOS 使用ProtocolBuffer进行数据传输](https://www.jianshu.com/p/bc39f33146b0)
* [iOS之ProtocolBuffer搭建和示例demo - TDX](http://www.cnblogs.com/tandaxia/p/6181534.html)
* [ProtocolBuffer在iOS中的使用](https://www.jianshu.com/p/751aa2b621d5)
* [Google Protocol Buffer 的优点与不足](https://www.jianshu.com/p/f44b84b08289)
* [我经历的 Protocol Buffers 那些坑 - gt9000的博客](https://blog.csdn.net/gt9000/article/details/86354510)
* [protocol buffer开发指南（官方） - 各位-请不吝赐教](https://blog.csdn.net/u013776188/article/details/77966092)
* [Google Protocol Buffer 的使用和原理](https://www.ibm.com/developerworks/cn/linux/l-cn-gpb/index.html)


#### Http & HTTPS

##### 抓包

* [Mac OS 下Charles+Chrome Omega配置方法 - New World](https://blog.csdn.net/liu251/article/details/52096142)
* [Charles抓https显示unknown解决方法](https://www.jianshu.com/p/498884193013)

##### HTTP

* [Socket_Interactive/ViewController.m](https://github.com/FieldsOfHope/Socket_Interactive/blob/master/Socket_Interactive/Socket_Interactive/ViewController.m)
* [socket编程实现HTTP请求 - 艾斯泽](https://www.cnblogs.com/icez/p/3983240.html)
<!--校验本地ssl-->
* [SSL Socket connection iOS](https://stackoverflow.com/questions/19232228/ssl-socket-connection-ios)

##### Https（仍在实践中）

1、 by CocoaAsyncSocket

* [iOS开发之Socket实现HTTPS GET请求通过Body传参](https://www.jianshu.com/p/4d7b8a0627a6)

2、 for Android

* [自己动手利用Socket 实现HTTP与HTTPS - 随手记两笔](https://blog.csdn.net/u013749540/article/details/73293921)
* [通过Socket进行HttP/HTTPS网页操作 - lianghugg](https://www.cnblogs.com/cxwx/archive/2011/10/25/2224105.html)

3、 Https 证书验证

* [iOS 的三种自建证书方法https请求相关配置 - t天t天](https://www.cnblogs.com/zhang6332/p/6289949.html)
* [iOS安全-证书、密钥及信任服务(Certificate, Key, and Trust Se...](https://www.jianshu.com/p/bf5da5dec233)
* [用于在iOS上运行HTTPS服务器的SSL身份证书](https://codeday.me/bug/20180927/264210.html)


#### 协议

* [TCP、UDP、HTTP、SOCKET、WebSocket之间的区别 - 火星男孩的分享空间](https://blog.csdn.net/sinat_31057219/article/details/72872359)
* [TCP/IP、Http、Socket的区别 - Pk_zsq的专栏](https://blog.csdn.net/Pk_zsq/article/details/6087367)
<!--删除下面博文关于 socket 的部分-->
* [5. Socket 编程](http://chenqihui.github.io/2018/09/05/iOS-%E7%BD%91%E9%80%9F%E7%9B%91%E6%8E%A7-%E5%AD%A6%E4%B9%A0%E7%AF%87/#Socket-%E7%BC%96%E7%A8%8B)


#### WebSocket

>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;WebSocket 和 Socket 虽然名称上很像，但两者是完全不同的东西， WebSocket 是建立在 TCP/IP 协议之上，属于应用层的协议，而 Socket 是在应用层和传输层中的一个抽象层，它是将 TCP/IP 层的复杂操作抽象成几个简单的接口来提供给应用层调用。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;正如描述的一样，WebSocket 是一个应用层协议，它定义了握手的规则，数据传输的格式，格式如下：

《[WebSocket 实现原理](http://zeeyang.com/2017/07/02/websocket/)》和
《[WebSocket协议简介](https://www.jianshu.com/p/3d2908d6efcd?utm_campaign=maleskine&utm_content=note&utm_medium=pc_all_hots&utm_source=recommendation)》里面都详细地介绍了各个字段的含义。

~~~
0                   1                   2                   3
0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-------+-+-------------+-------------------------------+
|F|R|R|R| opcode|M| Payload len |    Extended payload length    |
|I|S|S|S|  (4)  |A|     (7)     |             (16/64)           |
|N|V|V|V|       |S|             |   (if payload len==126/127)   |
| |1|2|3|       |K|             |                               |
+-+-+-+-+-------+-+-------------+ - - - - - - - - - - - - - - - +
|     Extended payload length continued, if payload len == 127  |
+ - - - - - - - - - - - - - - - +-------------------------------+
|                               |Masking-key, if MASK set to 1  |
+-------------------------------+-------------------------------+
| Masking-key (continued)       |          Payload Data         |
+-------------------------------- - - - - - - - - - - - - - - - +
:                     Payload Data continued ...                :
+ - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - +
|                     Payload Data continued ...                |
+---------------------------------------------------------------+
~~~

《[WebSocket的原理，以及和Http的关系](https://www.cnblogs.com/Herzog3/p/5088130.html)》不仅介绍了不同处，还相当趣味地介绍了 WebSocket 的握手规则，当然上面的博文也都有说明，和《[刨根问底HTTP和WebSocket协议](https://www.jianshu.com/p/0e5b946880b4)》、《[刨根问底HTTP和WebSocket协议(二）](https://www.jianshu.com/p/f666da1b1835)》

>二、WebSocket是什么样的协议，具体有什么优点。
>
>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;首先，相对于Http这种非持久的协议来说，WebSocket是一种持久化的协议。
>
>举例说明：
>
>（1）Http的生命周期通过Request来界定，也就是Request一个Response，那么在Http1.0协议中，这次Http请求就结束了。
>
>在Http1.1中进行了改进，是的有一个Keep-alive，也就是说，在一个Http连接中，可以发送多个Request，接收多个Response。
>
>但是必须记住，在Http中一个Request只能对应有一个Response，而且这个Response是被动的，不能主动发起。
>
>（2）WebSocket是基于Http协议的，或者说借用了Http协议来完成一部分握手，在握手阶段与Http是相同的。
>
>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;首先我们来看个典型的Websocket握手（借用Wikipedia的。。）
GET /chat HTTP/1.1  
Host: server.example.com  
Upgrade: websocket  
Connection: Upgrade  
Sec-WebSocket-Protocol: chat, superchat  
Sec-WebSocket-Version: 13  
Origin: http://example.com  
熟悉HTTP的童鞋可能发现了，这段类似HTTP协议的握手请求中，多了几个东西。  
我会顺便讲解下作用。  
Upgrade: websocket  
Connection: Upgrade  
>这个就是Websocket的核心了，告诉Apache、Nginx等服务器：注意啦，窝发起的是Websocket协议，快点帮我找到对应的助理处理~不是那个老土的HTTP。  
Sec-WebSocket-Key: x3JJHMbDL1EzLkh9GBhXDw==  
Sec-WebSocket-Protocol: chat, superchat  
Sec-WebSocket-Version: 13  
首先，Sec-WebSocket-Key 是一个Base64   encode的值，这个是浏览器随机生成的，告诉服务器：泥煤，不要忽悠窝，我要验证尼是不是真的是Websocket助理。  
然后，Sec_WebSocket-Protocol   是一个用户定义的字符串，用来区分同URL下，不同的服务所需要的协议。简单理解：今晚我要服务A，别搞错啦~  
>最后，Sec-WebSocket-Version 是告诉服务器所使用的Websocket Draft（协议版本），在最初的时候，Websocket协议还在 Draft 阶段，各种奇奇怪怪的协议都有，而且还有很多期奇奇怪怪不同的东西，什么Firefox和Chrome用的不是一个版本之类的，当初Websocket协议太多可是一个大难题。。不过现在还好，已经定下来啦~大家都使用的一个东西~ 脱水：服务员，我要的是13岁的噢→_→
>
>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;然后服务器会返回下列东西，表示已经接受到请求， 成功建立Websocket啦！  
HTTP/1.1 101 Switching Protocols  
Upgrade: websocket  
Connection: Upgrade  
Sec-WebSocket-Accept:   HSmrc0sMlYUkAGmm5OPpG2HaGWk=  
Sec-WebSocket-Protocol: chat  
这里开始就是HTTP最后负责的区域了，告诉客户，我已经成功切换协议啦~  
Upgrade: websocket  
Connection: Upgrade  
依然是固定的，告诉客户端即将升级的是Websocket协议，而不是mozillasocket，lurnarsocket或者shitsocket。  
>然后，Sec-WebSocket-Accept 这个则是经过服务器确认，并且加密过后的 Sec-WebSocket-Key。服务器：好啦好啦，知道啦，给你看我的ID CARD来证明行了吧。。
后面的，Sec-WebSocket-Protocol 则是表示最终使用的协议。
>
>至此，HTTP已经完成它所有工作了，接下来就是完全按照Websocket协议进行了。
其后是WebSocket协议的工作。

* [WebSocket 是什么原理？为什么可以实现持久连接？](https://www.zhihu.com/question/20215561)
* [WebSocket协议、SocketRocket源码学习](http://www.antyme.com/2016/12/14/WebSocket%E5%8D%8F%E8%AE%AE-SocketRocket%E6%BA%90%E7%A0%81%E5%AD%A6%E4%B9%A0/)
* [websocket与socket.io - 此生重演](https://www.cnblogs.com/cishengchongyan/p/6106602.html)
* [WebSocket和SocketIO总结 - foupwang](https://www.cnblogs.com/foupwang/p/7865694.html)


