---
title: iOS 网速监控 学习篇
date: 2018-09-05 15:45:02
tags:
  - HTTP
  - Socket
  - 网速
categories:
  - iOS
  
---
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;网络测速会在 App 性能里面经常提到。网络的好与坏影响了 App 的使用体验。因此监听网络状态，根据网络的状态来控制 App 的功能 或者 对应处理，能达到提升用户体验的目的，它也就成了 App 优化项的关键之一。当然性能还包括 App 体积，App 的热启动 & 冷启动 等等指标。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;先说下测速的需求，要在不增加额外的上传 & 下载的资源消耗，就能达到实时测速的目的。也就是得通过计算 App 现有业务的网络请求，获取的传输数据大小 & 请求时间，从而得出网速。

<!-- more -->

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;关于网络测速，在进行准备到测试开发的时候（5月末），居然同时间发现已经有[现成的文章](#jump) ，那就不重复写（还不是因为懒），内容原理差不多，甚至更详细。那就补充下准备的内容，后面的内容会有点偏离主题，不过都是为了网络请求准备了解的知识点：

#### 前文

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;网速关键就是：获取的传输数据大小 & 请求时间。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;通过 Charles 抓包工具，可以看到它有各种数值分析。其中就有 Size & Duration ，然后计算出 Speed。除了总的 Speed，它还区分了 Request Speed 和 Response Speed，也就是 Request & Response 有各自的 Data 和 Duration。

##### Size
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;它有 TLS Handshake、Header、Cookies、Body、Uncompressed Body、Compression。从这里，留意到我们要计算的数据，即 Body，在传输的时候是进过压缩，所以真正传输的 data 大小并不是接口访问后得到的数据包大小，不同的 encode 《[HTTP 协议中的数据压缩](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Compression)》，其压缩率都不同，从抓包的数据看出 gzip 大概的压缩率在70%到80%。还有其实传输的 data，虽然 Body 为主要占比，可是如果留意到 TLS Handshake，这是在 Duration 和 Size 都出现的。 

##### Duration

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;它包含了 DNS、Connect、TLS Handshake、Request、Response、Latency 的总和。 

##### TLS Handshake，

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;它其实就是使用的 HTTP 加入了验证 TLS / SSL 而形成 HTTPS。《[HTTP 与 HTTPS 的区别](http://www.mahaixiang.cn/internet/1233.html)》摘要：

>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;超文本传输协议HTTP协议被用于在Web浏览器和网站服务器之间传递信息，HTTP协议以明文方式发送内容，不提供任何方式的数据加密，如果攻击者截取了Web浏览器和网站服务器之间的传输报文，就可以直接读懂其中的信息，因此，HTTP协议不适合传输一些敏感信息，比如：信用卡号、密码等支付信息。
>
>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;为了解决HTTP协议的这一缺陷，需要使用另一种协议：安全套接字层超文本传输协议HTTPS，为了数据传输的安全，HTTPS在HTTP的基础上加入了SSL协议，SSL依靠证书来验证服务器的身份，并为浏览器和服务器之间的通信加密。
>
>一、HTTP和HTTPS的基本概念
>
>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;HTTP：是互联网上应用最为广泛的一种网络协议，是一个客户端和服务器端请求和应答的标准（TCP），用于从WWW服务器传输超文本到本地浏览器的传输协议，它可以使浏览器更加高效，使网络传输减少。
>
>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;HTTPS：是以安全为目标的HTTP通道，简单讲是HTTP的安全版，即HTTP下加入SSL层，HTTPS的安全基础是SSL，因此加密的详细内容就需要SSL。
>
>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;HTTPS协议的主要作用可以分为两种：一种是建立一个信息安全通道，来保证数据传输的安全；另一种就是确认网站的真实性。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;它也需要时间和数据交换，并且它不是每次都需要消耗，而是在频繁访问同个接口时候，它仍然处于链接的时候，是会简化数据，所以抓包发现它的时长有时长，有时短，数据包大小也是。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;使用《[SSL/TLS 握手过程详解](https://www.jianshu.com/p/7158568e4867)》进行握手之后，成功完成 HTTPS 的安全加密认证，后续的消息都将使用过程生成的秘钥进行加密，从而对 HTTP 的明文传输改为 HTTPS 的密文传输。接着就是 TCP 的握手《[TCP 为什么是三次握手，而不是两次或四次？](https://www.zhihu.com/question/24853633)》 & 《[面试题：三次握手、四次握手内容整理](https://blog.csdn.net/qq_18425655/article/details/52163228)》 & 《[三次握手](https://baike.baidu.com/item/%E4%B8%89%E6%AC%A1%E6%8F%A1%E6%89%8B/5111559?fr=aladdin)》。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;加密：《[互联网安全之数字签名、数字证书与PKI系统](https://www.jianshu.com/p/ffe8c203a471)》

##### <span id="jump">总结</span>

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;了解到这里，Duration 可以是个估计值，有点误差。而 iOS 通过网络得到的 Size 是还原了数据包的二进制大小，而非传输的数据大小，由于 HTTPS 的签名压缩率的不同，所以计算 Size 的误差就更大了，当时写到这里就停了下，直到发现 **小鱼周凌宇 的 [iOS 流量监控分析](http://zhoulingyu.com/2018/05/30/ios-network-traffic/#more)**，后续她还推荐了 [ImageSaveWarehouse/iOS 网络流量监控](https://github.com/zhshijie/ImageSaveWarehouse/blob/master/iOS%20%E6%96%87%E6%A1%A3/iOS%20%E7%BD%91%E7%BB%9C%E6%B5%81%E9%87%8F%E7%9B%91%E6%8E%A7/iOS%20%E7%BD%91%E7%BB%9C%E6%B5%81%E9%87%8F%E7%9B%91%E6%8E%A7.md) 。通过她的方式模拟传输的数据包大小，从而更精准地得出 Duration & Size 的获取，计算 iOS 网速，并进行实时监控。当时这种模拟 zip & signature 是消耗 CPU，如果真的需要，可以针对部分接口来计算。避免太大的开销。

#### 网络请求

##### NSURLConnection 

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;它是 iOS9 之前，最常使用的网络接口

~~~objc
+ (void)getCurrentNetIP {
    dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        NSString *rcUrl = @"https://www.xxxx.com";
        NSMutableURLRequest *request = [[NSMutableURLRequest alloc] initWithURL:[NSURL URLWithString:rcUrl] cachePolicy:NSURLRequestReloadIgnoringLocalCacheData timeoutInterval:8];
        [request setHTTPMethod:@"GET"];
        NSHTTPURLResponse* url_response = nil;
        NSError *error = nil;
        NSData *received = [NSURLConnection sendSynchronousRequest:request returningResponse:&url_response error:&error];
        if (received != nil) {
            NSString *aString = [[NSString alloc] initWithData:received encoding:NSUTF8StringEncoding];
            //...
        }
    });
}
~~~

##### NSURLSession & NSURLSessionDataTask 

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;它是 iOS9 之后苹果推行的，并且 AFNetwork 3.0 版本也更新使用。

~~~objc
NSURL *url = [NSURL URLWithString:urlString];
NSURLSession *session = [NSURLSession sharedSession];

NSURLSessionDataTask *dataTask = [session dataTaskWithURL:url completionHandler:^(NSData * _Nullable data, NSURLResponse * _Nullable response, NSError * _Nullable error) {
    if (error == nil && data.length > 0) {
        NSDictionary *dict = [NSJSONSerialization JSONObjectWithData:data options:NSJSONReadingAllowFragments error:nil];
    }
    else {
        NSString *errorString = @"";
    }
}];
[dataTask resume];
~~~

参考：  

* [NSURLSession 与 NSURLConnection区别](https://www.jianshu.com/p/056b1817d25a)

##### NSOperation & NSOperationQueue 
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;上述两个都有下载 & 上传文件的接口，而 Operation 也可以针对文件进行下载。这一部分其实早期写开发下载模块的时候有使用过。

参考：

* [NSOperation和NSOperationQueue相关](https://www.jianshu.com/p/4443d668e931)  
* [iOS多线程之NSOperationQueue](https://www.jianshu.com/p/52fe1b85c404)  

##### GCDWebServer 

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;本地服务，可用于 task 一边下载视频文件，然后开启本地服务，为播放器提供本地的视频流，从而达到一边下载一边播放的功能。[TODO]
 
参考： 

* [GCDWebServer](https://www.jianshu.com/p/534632485234)

#### NSURLProtocol

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;以上是 iOS 的网络请求提供的高级接口和对象。它们能满足我们客户端在请求获取数据。如前文说说，真正的 Speed 其实按开头的想法计算是不准确的，有很大的误差，但是换个角度，如果 App 内部使用同一套规则计算，其实它的网速快慢就是相对的，所以它对于 Charles 的数据的“误差”也就可以忽略。只要我们按获取的数据包大小比请求到消耗的时间来定义快慢标准就可以了。因此，继续这个话题引用 NSURLProtocol 《[iOS H5容器的一些探究（二）：iOS下的黑魔法NSURLProtocol](https://www.jianshu.com/p/03ddcfe5ebd7)》，使用它其实等同于在每个请求的接口 task 的回调来计算是一样的，它的优点是覆盖了所有的基于系统的 NSURLConnection 或者 NSURLSession 进行封装的网络请求，包括 UIWebView 。

~~~objc
- (void) connectionDidFinishLoading:(NSURLConnection *)connection {
	CGFloat endTime = CFAbsoluteTimeGetCurrent();
	CGFloat useTime = endTime - self.startTime;
	NSLog(@"chen--useTime--%f", useTime);

	CGFloat sizeB = connection.currentRequest.HTTPBody.length + self.totalSize;
	CGFloat kb = sizeB/1024.f;
	NSLog(@"chen--data--%f", kb);

	NSLog(@"chen--speed--%f", kb/useTime);

    [self.client URLProtocolDidFinishLoading:self];
}
~~~

#### 其他网络测试

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;说起网络测试，肯定看过 Reachability 和 SimplePing，这两个都是苹果开源的工具。Github 上有多个扩展版本。

##### Reachability

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[Reachability](https://developer.apple.com/library/content/samplecode/Reachability/Introduction/Intro.html) 就不在细说，它可以检查当前网络 和 实时监控网络的切换情况，自己的项目也是针对它再次修改，细化到判断当前网络的类型，如 4G、3G 等。

~~~objc
// 开启
- (void)p_startNetReachable:(BOOL)bEable {
    dispatch_async(dispatch_get_main_queue(), ^{
        if (bEable == YES) {
            self.reachability = [QHReachability reachabilityForInternetConnection];
            if (self.currentStatus != [self.reachability currentReachabilityStatus]) {
                self.currentStatus = [self.reachability currentReachabilityStatus];
                [self p_netReachable:self.currentStatus];
            }
            [self.reachability startNotifier];
        }
        else {
            if (self.reachability != nil) {
                [self.reachability stopNotifier];
                self.reachability = nil;
            }
        }
    });
}
// 监听
// 添加
[[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(qhReachabilityChangedAction:) name:kQHReachabilityChangedNotification object:nil];
// 移除
[[NSNotificationCenter defaultCenter] removeObserver:self name:kQHReachabilityChangedNotification object:nil];
// Action
- (void)qhReachabilityChangedAction:(NSNotification *)notif {
    QHReachability *curReach = [notif object];
    NetworkStatus status = [curReach currentReachabilityStatus];
    if (self.currentStatus == status)
        return;
    self.currentStatus = status;
    [self p_netReachable:self.currentStatus];
}
~~~

##### SCNetworkReachability  

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;文章讲解了内部都是使用 [SCNetworkReachability](https://developer.apple.com/documentation/systemconfiguration/scnetworkreachability?language=objc)《[用SCNetworkReachability判断联网状态](https://blog.csdn.net/yyyyccll/article/details/68942015)》 来实现的。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;也可以使用《[iOS Reachability检测网络状态](https://www.cnblogs.com/sunfuyou/p/6838609.html)》中介绍了 AFNetworkReachabilityManager。

~~~objc
#import "NetworkUtil.h"
#import <SystemConfiguration/SCNetworkReachability.h>
#import <netdb.h>

@implementation NetworkUtil

+ (BOOL)isNetworkReachable {
    // Create zero addy
    //创建零地址，0.0.0.0的地址表示查询本机的网络连接状态
    struct sockaddr_in zeroAddress;
    bzero(&zeroAddress, sizeof(zeroAddress));
    zeroAddress.sin_len = sizeof(zeroAddress);
    zeroAddress.sin_family = AF_INET;

    // Recover reachability flags
    // SCNetworkReachabilityFlags：保存返回的测试连接状态
    // 其中常用的状态有：
    // kSCNetworkReachabilityFlagsReachable：能够连接网络
    // kSCNetworkReachabilityFlagsConnectionRequired：能够连接网络，但是首先得建立连接过程
    // kSCNetworkReachabilityFlagsIsWWAN：判断是否通过蜂窝网覆盖的连接，比如EDGE，GPRS或者目前的3G.主要是区别通过WiFi的连接。
    SCNetworkReachabilityRef defaultRouteReachability = SCNetworkReachabilityCreateWithAddress(NULL,
        (struct sockaddr*)&zeroAddress);
    SCNetworkReachabilityFlags flags;

    BOOL didRetrieveFlags = SCNetworkReachabilityGetFlags(defaultRouteReachability, &flags);
    CFRelease(defaultRouteReachability);

    if (!didRetrieveFlags) {
        return NO;
    }

    BOOL isReachable = flags & kSCNetworkFlagsReachable;
    BOOL needsConnection = flags & kSCNetworkFlagsConnectionRequired;
    return (isReachable && !needsConnection) ? YES : NO;
}

@end
~~~

##### SimplePing

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;常用的是 [STSimplePing.h](https://github.com/lovesunstar/STPingTest/blob/master/STPingTest/Classes/PingTester/STSimplePing.h) 这个版本，它扩展了一个每个 syn 之后响应所消耗的时间。通过它我们也可以判断当前网络的状态。但是回顾测试的需求，它是消耗额外资源的方式，也不建议实时使用。它通过 ping 下域名的延迟及时长来分析网络状态。

~~~objc
self.simplePing = [[STSimplePing alloc] initWithHostName:@"www.baidu.com"];
self.simplePing.delegate = self;
[self.simplePing start];
    
#pragma mark - STSimplePingDelegate

- (void)st_simplePing:(STSimplePing *)pinger didReceivePingResponsePacket:(NSData *)packet timeToLive:(NSInteger)timeToLive sequenceNumber:(uint16_t)sequenceNumber timeElapsed:(NSTimeInterval)timeElapsed {
    [self.simplePing stop];
    NSLog(@"%s", __FUNCTION__);
    NSLog(@"simplePing==%f", timeElapsed);//0.078942
}
~~~

参考：

* [iOS开发中WiFi相关功能总结](https://www.jianshu.com/p/8471b68203e8)
* [iOS 如何进行网络测速](https://www.jianshu.com/p/9f67b7716b9d)

##### 网卡

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;上述参考中还记录了获取其他数据的功能及不仅仅使用 SimplePing，还使用网卡来进行计算。网卡的这部分只做记录，因为在计算网速上，它无法测出满载，即当前设备网络的真正网速。它的用途更在于记录消耗的流量。 

~~~objc
- (long long)getDeviceCurrentBytesCount {
    struct ifaddrs* addrs;
    const struct ifaddrs* cursor;

    long long currentBytesValue = 0;
    
    if (getifaddrs(&addrs) == 0) {
        cursor = addrs;
        while (cursor != NULL) {
            const struct if_data* ifa_data = (struct if_data*)cursor->ifa_data;
            if (ifa_data) {
                // total number of octets received
                int receivedData = ifa_data->ifi_ibytes;
                
                currentBytesValue += receivedData;
            }
            cursor = cursor->ifa_next;
        }
    }
    freeifaddrs(addrs);
    
    NSLog(@"BytesCount:%lld",currentBytesValue);
    return currentBytesValue;
}
~~~

##### tcpdump

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;它在 [GT](http://gt.qq.com/) 中使用过。GT 是腾讯开源的项目，主要是针对 APP 实时的性能测试记录，包含 CPU、电池等消耗。而其中也包含了网络测速，它使用的就是本次要说明的 tcpdump ，文档其实也就简单描述
> tcpdump 抓 包 手 机 需 要 越 狱 ， 且 暂 时 只 支 持 iOS6 。 抓 包 保 存 目 录 ：
Documents/GT/Plugin/pcap/

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;并没有将 tcpdump 的代码开源处理，因此无法使用。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;tcpdump《[抓包工具tcpdump用法说明](https://www.cnblogs.com/f-ck-need-u/p/7064286.html)》，它其实是 PC 系统常用的抓包命令《[通过tcpdump对iOS进行流量分析（无需越狱）
](https://www.jianshu.com/p/b062c1dc2f0e)》里面介绍了结合 PC 工具进行抓包，而越狱的使用是通过在 cydia 里安装 openssh，tcpdump 两个工具之后使用手机终端操作，而 App 开发调用的方法暂无资料《[利用tcpdump抓取ios的tcp数据包](https://blog.csdn.net/skylin19840101/article/details/38270053)》。


##### Socket 
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;这部分在 《iOS Reachability检测网络状态》里面最后写了。其实这里不想过多的介绍，Socket 这部分后续会开个新章节来说明。

~~~objc
#import <arpa/inet.h>

/// 服务器可达返回true
- (BOOL)socketReachabilityTest {
    // 客户端 AF_INET:ipv4  SOCK_STREAM:TCP链接
    int socketNumber = socket(AF_INET, SOCK_STREAM, 0);
    // 配置服务器端套接字
    struct sockaddr_in serverAddress;
    // 设置服务器ipv4
    serverAddress.sin_family = AF_INET;
    // 百度的ip
    serverAddress.sin_addr.s_addr = inet_addr("202.108.22.5");
    // 设置端口号，HTTP默认80端口
    serverAddress.sin_port = htons(80);
    if (connect(socketNumber, (const struct sockaddr *)&serverAddress, sizeof(serverAddress)) == 0) {
        close(socketNumber);
        return true;
    }
    close(socketNumber);;
    return false;
}
~~~

#### Socket 编程

##### Socket  

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;说到 iOS 的 Socket 编程，想起最早看到，或者是说第一个组长开发的。使用的就是苹果封装的 CFNetwork 加 C 语言编写的 Socket 通用 API，包含 CFReadStream、CFWriteStream等。其实要找代码才能确认对不对，可以参考 [CocoaAsyncSocket](https://github.com/robbiehanson/CocoaAsyncSocket) 这里面包含异步和 udp实现的 socket 和 《[ios底层Socket编程理解](https://www.jianshu.com/p/1883977690b3)》，当然可以自己直接调用。

>Socket:又称”套接字”,应用程序通过”套接字”向网络发送请求或应答,它是一个针对TCP和UDP编程的接口，借助它建立TCP/UDP连接。socket连接就是所谓的长连接,理论上客户端和服务器端一旦建立起连接将不会主动断掉.

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;\<sys/socket.h> 这是 C语言 Socket 通用库，即支持 iOS，也支持 Mac。CocoaAsyncSocket 里面都有例子。

~~~
//创建 socket，SOCKET_STREAM 是实现 TCP，SOCKET_DGRAM 则是实现 UDP
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

~~~
TCP 服务器端：  
socket() -> bind() -> listen() -> accept() -阻塞直到有客户端连接-> read() -> write() -> read() -> close()  
创建套接字 -> 绑定本机端口 -> 侦听端口 -> 建立连接 - 等待客户端连接 -> 数据传输（请求、回应、请求数据 -> 结束连接  
TCP 客户端：  
socket() -> connect -> write() -> read() -> close()  
创建套接字 -> 建立连接 -> 数据传输（回应、请求数据） -> 结束连接  
~~~

相应的参数说明：  
《[socket编程需要哪些头文件](https://www.cnblogs.com/cucumber/p/3922756.html)》和《[socket创建流程及代码示例](https://blog.csdn.net/zzk00007/article/details/75041243)》

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;不过常使用下面这几个知名的第三方 Socket 库，分别是 facebook 开源的
[SocketRocket](https://github.com/facebook/SocketRocket)、针对 SocketRocket 进一步业务封装的 NetEase [pomelo](https://github.com/NetEase/pomelo)，而网易的 pomelo 也是目前直播间公屏及广播，它自家用用于手游等游戏的 Socket 服务。另外还有使用 [socketio/socket.io](https://github.com/socketio/socket.io)，SocketIO 其实是将 WebSocket、AJAX和其它的通信方式全部封装成了统一的通信接口。并且上述说的第三方库，里面都有 WebSocket 的代码实现，WebSocket 是基于 TCP 的实时通讯，如今在 PC，移动端，H5 都有兼容。而项目使用的 Socket 其实也就是使用 WebSocket。

##### WebSocket

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

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;而 WebSocket 的接口也是否清晰，看 Demo 就好。WebSocket 的心跳包？

（待续...）




