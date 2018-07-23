---
title: iOS 字符的 Encode & Decode
date: 2018-07-23 23:08:53
tags: 
  - String
categories:
  - iOS
  
---
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;本次的话题应该算是老话题。在开发用户系统，用户昵称修改，聊天等功能的时候，会遇到用户输入 Emoji，特殊字符。甚至直接输入 Unicode 编码的时候。如果客户端提交给服务端，再从服务端获取，直接显示可能会出现乱码。甚至有时是服务端进行编码保存，然后给客户端，并没有协商谁来负责解码。这都只是一方面，因为特殊字符的存在还会影响数据上报。当服务端拿到这种特殊数据，会无法解析，导致请求失败等等。所以需要对字符进行 Encode & Decode。

<!-- more -->

#### 换行

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"\r\n" 这是个换行符，也是比较常见出现问题的符号。如上几期的心愿飞屏，用户填写心愿时是可以输入换行，无论在客户端还是H5，然后当飞屏获取的心愿包含这个换行符的时候，就会出现异常，尤其需要通过计算文本的宽高来设置飞屏的 frame，造成样式变形，文本显示不全。因此需要将 "\r\n" 替换成空字符串，如下两种实现：

```swift
// 1、直接替换
let s1 = "\r\n"
let s2 = s1.replacingOccurrences(of: "\r\n", with: "")

// 2、使用正则表达式替换
extension String {
    var count: Int {
        let string_NS = self as NSString
        return string_NS.length
    }
    
    func pregReplace(pattern: String, with: String,
                     options: NSRegularExpression.Options = []) -> String {
        let regex = try! NSRegularExpression(pattern: pattern, options: options)
        return regex.stringByReplacingMatches(in: self, options: [],
                                              range: NSMakeRange(0, self.count),
                                              withTemplate: with)
    }
}

let s3 = "\n\r\nhellp\n"
let s4 = str1.pregReplace(pattern: "\r|\n", with: "")
```

#### URL

>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;百分号编码（英语：Percent-encoding）, 也称作URL编码（英语：URL encoding）, 是特定上下文的统一资源定位符 (URL)的编码机制. 实际上也适用于统一资源标志符（URI）的编码。也用于为"application/x-www-form-urlencoded" MIME准备数据, 因为它用于通过HTTP的请求操作(request)提交HTML表单数据。出自《维基百科，自由的百科全书》

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;正如维基百科里面描述的，URLEncode 是把传送中的 String 或者 请求的入参 String 里面的中文和特殊字符进行指定编码（UTF-8、GBK等，中文在不同编码下的结果不同）。因为是这种字符会破坏 URL 的结构，导致服务端和浏览器无法正常识别。因此通过 URLEncode 处理后使其能够正常传输，其作用就是使此 url 合法化。

```swift
extension String {
    func urlEncoded() -> String {
        let charactersToEscape = "?!@#$^&%*+,:;='\"`<>()[]{}/\\| "
        let allowedCharacters = CharacterSet(charactersIn: charactersToEscape).inverted
        let encodeUrlString = self.addingPercentEncoding(withAllowedCharacters: allowedCharacters)
//        let encodeUrlString = self.addingPercentEncoding(withAllowedCharacters: .urlQueryAllowed)
        return encodeUrlString ?? ""
    }
    
    func urlDecoded() -> String {
        return self.removingPercentEncoding ?? ""
    }
}
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;除了自定义需要编码的字符，也可以使用苹果提供的 NSCharacterSet 集合，如下：

```objc
URLHostAllowedCharacterSet      "#%/<>?@\^`{|}
URLFragmentAllowedCharacterSet  "#%<>[\]^`{|}
URLPasswordAllowedCharacterSet  "#%/:<>?@[\]^`{|}
URLPathAllowedCharacterSet      "#%;<>?[\]^`{|}
URLQueryAllowedCharacterSet     "#%<>[\]^`{|}
URLUserAllowedCharacterSet      "#%/:<>?@[\]^`
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;保留字符的百分号编码如下:

| ! | # | $ | & | ' | ( | ) | * | + | , | / | : | ; | = | ? | @ | [ | ] |   
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |   
| %21 | %23 | %24 | %26 | %27 | %28 | %29 | %2A | %2B | %2C | %2F | %3A | %3B | %3D | %3F | %40 | %5B | %5D |   


#### HTML

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;HTML 字符实体，它是是以 "&" 开头和 ";"结尾的字符。它会出现在服务端传给客户端数据时或者是打印 HTML 网页的文本时，特别是服务端为了 防XSS攻击 时，就一定会 encode，如果它传回来没有 decode，那客户端就会出现乱码。此时就需要使用 UTF8编码 将 HTML 实体中的字符串转换为正确的字符表示形式。例如: "&hellip;" 所代表的省略实体被转换为 "…" 这个省略号。

实现方式：  

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;一般使用字典记录下对应的字符与实体名称，然后通过该字典进行匹配来转换，从而实现 encode & decode。

[DTCoreText-Demo/NSString+HTML.m](https://github.com/RobinChao/DTCoreText-Demo/blob/0d6b09fde1d305cef1bf17e2f6872fe755d8a9d6/Test/Pods/DTCoreText/Core/Source/NSString%2BHTML.m)

字符对应的实体名称参考：《[HTML ISO-8859-1 参考手册](http://www.w3school.com.cn/tags/html_ref_entities.html)》

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;通过如下转换的 NSAttributedString ，获取 String 值也可以实现 decode。

```objc
// 字符串转富文本
+ (NSString *)strToAttriWithStr:(NSString *)htmlStr{
   NSAttributedString *attr = [[NSAttributedString alloc] 
							   initWithData:[htmlStr 
							   dataUsingEncoding:NSUnicodeStringEncoding] 
							   options:@{NSDocumentTypeDocumentAttribute:NSHTMLTextDocumentType} 
							   documentAttributes:nil 
							   error:nil];
   return attr.string;
}
```

#### Emoji

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Emoji 的需求是很常见，很多如名称，信息填写都是不支持 Emoji 的，因为有时传输 Emoji 时，服务端保存的是 Unicode 编码，再传回其他客户端可能出现乱码（即直接显示 Unicode 了）。这里一般限制会在客户端，即检查填入的信息是否包含字符。在 iOS 里，标准的 Emoji 经过传输是可以一致的，即不会因为传输时转码造成错误。而其他 iOS 自定义或者输入法新增的 Emoji 就很难保证。所以有些限制还是需要的。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;如下是标准 Emoji 的 Unicode 区间，即将字符转换为十六进制的 Unicode 编码，在如下区间就表示该字符是 Emoji。

《[Emoji 的编码以及常见问题处理](https://segmentfault.com/a/1190000007594620)》

```
<U+1F300> - <U+1F5FF>      # symbols & pictographs
<U+1F600> - <U+1F64F>      # emoticons
<U+1F680> - <U+1F6FF>      # transport & map symbols
<U+2600>  - <U+2B55>       # other
```

《[过滤 NSString 中的 Emoji](https://blog.csdn.net/kebing1011/article/details/46868197)》里的实现：

```objc
- (BOOL)isNotEmoji:(UInt64) codePoint {
    return (codePoint == 0x0)
    || (codePoint == 0x9)
    || (codePoint == 0xA)
    || (codePoint == 0xD)
    || ((codePoint >= 0x20) && (codePoint <= 0xD7FF))
    || ((codePoint >= 0xFF00) && (codePoint <=
                                  0xFFFF));
}
```

#### Unicode

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;说到这里，如果回看上述提到 URL、 HTML、 Emoji，其字符对应的编码都是定义在字典里面。Unicode 就是它们的通用表，也就是说它们就是整个 Unicode 表的其中一块。它们的 encode & decode 其实也都基于对应的字典来进行，不同之处在于使用的字典是系统提供还是需要额外提供。

需要简单介绍下常用的字符编码：  

* ASCII 定义了常用 0 - 127 （0x007F） 号字符，128 - 255（0x00FF） 号字符 被不同国家自定义。  
* Unicode 定义了 21 位（从 U+0000 到 U+10FFFF），提供了 1,114,112 个码点，每个码点对应一个字符，并构成编码空间。而编码空间被分成 17 个平面（plane）（0~16号平面），每个平面有 65,536 个字符。其中，0 号平面叫做「基本多文种平面」（Basic Multilingual Plane, BMP）。
* UTF-8、UTF-16、UTF-32 就是基于 Unicode 的编码方式。

##### UTF-8  
编码规则如下：  

| Unicode 十六进制码点范围 | UTF-8 二进制 |   
| --- | --- |   
| 0000 0000 - 0000 007F | 0xxxxxxx |   
| 0000 0080 - 0000 07FF	 | 110xxxxx 10xxxxxx |   
| 0000 0800 - 0000 FFFF	 | 1110xxxx 10xxxxxx 10xxxxxx |   
| 0001 0000 - 0010 FFFF	 | 11110xxx 10xxxxxx 10xxxxxx 10xxxxxx |   

##### UTF-16 以及「代理对」（Surrogate Pairs）

编码规则如下：

| UTF-16(二进制)  | 编码点（二进制） | 范围 |   
| --- | --- | --- |   
| xxxxxxxxxxxxxxxx | xxxxxxxxxxxxxxxx | U+0000 –- U+FFFF |   
| 110110xxxxxxxxxx 110111yyyyyyyyyy | xxxxxxxxxxyyyyyyyyyy + 0x10000 | U+10000 –- U+10FFFF |   

>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;但是，通常人们谈到 UTF-16是因为它涉及到了一个在编码点术语中被称作“代理（surrogate）”的东西。所有在范围 U+D800-U+DFFF（或在其他范围） 中的编码点，这些和上表中二进制前缀 110110 和 110111 匹配的编码点——是 UTF-16 中的保留区域，它们自身不表示任何有效的字符。它们仅用于上面 2 个字的编码模式中，被称作“代理对（surrogate pair）”，代理编码点在任何其他情况下都是非法的！它们不能出现在 UTF-8 和 UTF-32 中。

参考文章：

* [彻底弄懂 Unicode 编码](https://blog.whezh.com/encoded/)
* [ObjC 中国 - NSString 与 Unicode](https://objccn.io/issue-9-1/)  
* [写给程序员的 Unicode 入门介绍](http://blog.jobbole.com/111261/)
* [年轻人，Emoji是这样控制了你的-虎嗅网](https://www.huxiu.com/article/163386.html)
* [iPhone emoji 问题牵出的 Unicode 代理区的思考](https://blog.csdn.net/hherima/article/details/38961575)

#### Length

《[字符串的长度，是字符数量，还是字节数量？](http://www.cnblogs.com/ljhdo/p/4546081.html)》

#### 数字

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;将字符串转换数字：

```swift
open class func StringToInt(_ str: String) -> Int {
    let string = str
    var cgInt: Int = 0
    
    if let doubleValue = Double(string) {
        cgInt = Int(doubleValue)
    }
    return cgInt
}
```