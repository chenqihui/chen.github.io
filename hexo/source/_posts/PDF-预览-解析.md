---
title: PDF 预览 & 解析
date: 2018-08-16 18:08:53
tags:
  - Swift
  - PDF
  - MacOS
categories:
  - 工具
  
---
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;简单讲就是它独立包含文档所有信息，如文本、字体、图形等等。并且当看到下面解析工具解析看到结构，每一页也都具有完整信息，这样就可以分开处理显示。可以通过 [可移植文档格式 - 维基百科，自由的百科全书](https://zh.wikipedia.org/wiki/%E5%8F%AF%E7%A7%BB%E6%A4%8D%E6%96%87%E6%A1%A3%E6%A0%BC%E5%BC%8F) & [pdf_百度百科](https://baike.baidu.com/item/pdf/317608?fr=aladdin) 来了解更多 PDF 信息。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;回到需求，产品上看可能 APP 只要能看，并在看到第几页的时候显示相应的提示或者其他动作。而对于开发者，如果使用自定义的话，会疑问如何显示？是列表呢、还是翻页，加载效果？加载方式？在线还是本地等等。并且自定义显示的效率如何？是否占内存，占CPU。是否需要编辑？再深入就是研究 PDF 结构。然后发现想多了，还是老老实实先实现产品需求。

<!-- more -->

#### 工具

1、[QHPDFDemo](https://github.com/chenqihui/QHPDFDemo)：
  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;iOS 实现预览 PDF。主要使用 自定义绘制 PDFView。用 UIScrollView 或者UIPageViewController 两种容器来显示 PDF。UIScrollView 是模仿 UIWebView 的样式，而 UIPageViewController 拥有翻页功能，并可以设置不同翻页的效果。具体实现可以查看源码。

2、[PDF-Voyeur](https://github.com/chenqihui/PDF-Voyeur)   

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Mac 解析 PDF 结构，参考《[提取pdf目录的方法](http://phaibin.tk/2012/01/06/ti-qu-pdfmu-lu-de-fang-fa)》里面的一个 Github 项目，看注释应该是苹果官方的 Example，但是苹果官方已经查找不到对应的项目，所以 fork 下来修改并运行起来，只是几个 API 修改。其主要使用苹果提供的 C库 API 获取 PDF 信息。而 PDF 的结构需要阅读 Adobe 制定的标准《[PDF32000_2008](https://www.adobe.com/content/dam/acom/en/devnet/pdf/pdfs/PDF32000_2008.pdf)》

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**\*就不截图了，以上工具可跑 Demo 看哈**

#### QHPDFDemo 的开发之路

| 记录 |  
| --- |  
|1、首先想到的是通过 UIScrollView 来绘制整个 PDF，UIScrollView 自带 捏合 功能，做完感觉完美了。|  
|2、开开心心开发 page 模式，即用 UIPageViewController 来实现，添加 enum 来实现控制。做到这里真的还蛮顺利的。page 模式不是需要重点就没有再添加 捏合 功能，后续需要再开发。|  
|3、接着实现 UIWebView 的显示 PDF，看下面就发现，本以为可以作为在线预览的模式，可是由于单 PDF，不清楚 size 大小，因此需要缓存才能拿到。略坑，但是如果服务端在接口范围依然能在线预览。由于样式固定也就先到这里。|  
|4、依然很顺利。但是想到用的 PDF 只有 10 页，改成 几百页 的时候，Scroll 模式由于全绘制，直接卡住，并且还报了 size [问题](#问题 2)，还有 page 模式也在快速翻页出现警告[问题](#问题 1)，page 的添加动画完成控制解决，还算 OK。|  
|5、而 scroll 模式的问题，想着进行局部绘制，并动态修改 父view 的 frame，不在一开始就设置最大（因为 contentSize 达到 pdfHeight * pdfCount，几百页时候达到 几十万 px），局部绘制的时候由于惯性思维，一直想着在同一个 父View 上绘制（只需设置好 y），但是发现 CGContext 下绘制效果不好，控制不好局部刷白绘制及 CTM，总影响到前面已经绘制的。|  
|6、傻了傻了，既然是这种，为什么不在 父view 上添加 子View 再绘制，这样坐标也可以忽略。没错，这样就简单多了，再想想，这不就是 UICollectionView 了，用系统的回收机制效率还更好。新增 Collection 模式，开发完成后，嗯，简单。问题又来了，缩放问题咋整？|  
|7、UICollectionView & UITableView 本身是 UIScrollView 的子类，但是其 ZoomScale 没法用。只有在整个添加 UIScrollView 或者在 Cell 添加 UIScrollView。无论哪种都有瑕疵。整个添加 UIScrollView，当其放大状态，UIScrollView 就会拦截手势，滚动区域就成了 UIScrollView 的 ContentSize，无法回到 UICollectionView。而通过在 zoomEnd 回调来设置 ContenSize.height = 0，可以回到 UICollectionView 的滚动，只是这样修改放大时的显示固定了（即不是捏合位置，会有抖动，体验不好）|  
|8、而 子view 添加 UIScrollView ，这部分还没实现，不过这部分 Demo 比较多，苹果官方也给出了（地址找不到了，尴尬），类似这篇 [添加夹点缩放到 UICollectionView](https://www.itstrike.cn/Question/f497e47b-bd8f-4b64-9d15-36d8e0869642.html) 通过实行 cell 的捏合控制 layout 的 itemSize，可是大部分在 手势end 时候都还原了，而没有还原的 [Alua-Kinzhebayeva/iOS-PDF-Reader: PDF Reader for iOS written in Swift](https://github.com/Alua-Kinzhebayeva/iOS-PDF-Reader) 这种，只能一页页放大。 |  
|9、还是没达到需求，最终回到 scroll 模式，然后实行类似 UICollectionView 的页面加载，Demo 里面是加载固定区域，区域外的 子view 会回收，测试过如果 Demo 里的 pdf 加载到 100 页是内存爆了，直接杀进程了。具体实现可以查看代码|  
|10、优化空间：加载的渐进式显示，快速滑动判断来控制添加 PDF page，绘制的优化（目前实现的 CGContext 绘制 PDF），等等。总之，坑还是蛮多的。功能还可以添加编辑。|  
||  


----


PDF文档预览的几种方式，内容可以参考：

* QLPreviewController
* UIDocumentInteractionController
* UIWebView
* CGContexDrawPDFPage

#### QLPreviewController & UIDocumentInteractionController

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;它们俩都是苹果封装好的文件预览控制器，可以预览多种文件格式，如PDF、doc、jpg、key等等。并且提供了系统分享功能，及 当前页码/总页码 的显示。差别是 QL 是一个 UIViewController。而 UIDocumentInteractionController 本身并不是一个控制器类，它直接继承 NSObject，所以就不能直接 push 或者模态跳转了，所以需要使用它类方法提供的模态跳转函数。QL 能支持多文件预览（即提供 DataSource 切换预览文件的 URL）和横滑切换，而 UIDocumentInteractionController 就不支持，只能一次显示一个。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;***而它们的缺点都是只能预览本地文件。***

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;它们的使用可参考：[QLPreviewController 文件快速浏览](https://www.jianshu.com/p/ad9544915877)  

#### UIWebView  

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;它不仅能预览多种文件，还支持在线与本地两种。可以说 UIWebView 真的很强大。如果使用 UIWebView 预览 PDF，会发现没有总页码，也不知当前页码是多少，因此需要而外的解析 PDFDocument。

##### 1、获取 PDF 的信息（如总页数）

```swift
if let document = CGPDFDocument(url as CFURL) {
	count = document.numberOfPages
}
```

##### 2、计算当前页码：

```swift
// MARK: - UIWebViewDelegate
 
// 1、用 UIScrollView 的 contentSize.height 除以 总页码，算出每页的 height

func webViewDidFinishLoad(_ webView: UIWebView) {
    pdfPageHeight = webView.scrollView.contentSize.height / CGFloat(count)
}
    
// MARK: - UIScrollViewDelegate
    
// 2、监听 UIScrollView 的滑动事件获取滑动偏移值 contentOffset.y 来计算当前的页码

func scrollViewDidScroll(_ scrollView: UIScrollView) {
    guard pdfPageHeight > 0 else {
        return
    }
    let y = scrollView.contentOffset.y
    //        // 以页面 bottom 到达屏幕底部为翻页
    //        let currentIndex = ((y + self.frame.size.height) / scrollView.zoomScale) / pdfPageHeight
    // 以页面 top 到达屏幕顶部为翻页
    let currentIndex = Int((y / scrollView.zoomScale) / pdfPageHeight) + 1
    
    dataSource?.showInPDFPage(view: self, index: currentIndex)
}
```

##### 3、注意

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;很明显，如果想通过 PDF 结构得到总页码来计算页码的话，需要整个 PDF 的数据，也就是变相的为实现此功能，等同于读本地 PDF 一样。除非获取 PDF 的时候，服务端额外提供 PDF 的页码。

 
#### QHPDFView + CGPDFDocument


参考：

* [Franzeyang/PDFBrowse](https://github.com/Franzeyang/PDFBrowse)
* [iOS开发笔记——PDF的显示和浏览](https://blog.csdn.net/yiyaaixuexi/article/details/7645725)

CATiledLayer

* [ios 超大图显示：CATiledLayer的使用，关于tileSize的用法 - 简书](https://www.jianshu.com/p/ee0628629f92)

##### 问题 1

UIPageViewController 在快速翻页的时候，当动作没有完成时就翻另一页，控制台会提示：
>'Unbalanced calls to begin/end appearance transitions for 

* [ios - UIPageViewController transition 'Unbalanced calls to begin/end appearance transitions for ' - Stack Overflow](https://stackoverflow.com/questions/13248282/uipageviewcontroller-transition-unbalanced-calls-to-begin-end-appearance-transi)  

解决：使用的是 pageIsAnimating 第一个解答的方案，如下：

```swift
// 标识翻页动画是否正在进行中
var pageIsAnimating: Bool = false

// MARK - UIPageViewControllerDelegate
    
func pageViewController(_ pageViewController: UIPageViewController, willTransitionTo pendingViewControllers: [UIViewController]) {
    pageIsAnimating = true
}
    
func pageViewController(_ pageViewController: UIPageViewController, didFinishAnimating finished: Bool, previousViewControllers: [UIViewController], transitionCompleted completed: Bool) {
    if finished == true, completed == true {
        pageIsAnimating = false
    }
}
    
private func p_goPage(currentViewController: QHPageViewController, incrementIndex: Int) -> QHPageViewController? {
    if pageIsAnimating == true {
        return nil
    }
    ...
    return vc
}

// MARK - UIPageViewControllerDataSource
    
func pageViewController(_ pageViewController: UIPageViewController, viewControllerBefore viewController: UIViewController) -> UIViewController? {
    return p_goPage(currentViewController: viewController as! QHPageViewController, incrementIndex: -1)
}
    
func pageViewController(_ pageViewController: UIPageViewController, viewControllerAfter viewController: UIViewController) -> UIViewController? {
    return p_goPage(currentViewController: viewController as! QHPageViewController, incrementIndex: 1)
}
```

##### 问题 2

UIScrollView 在初始高度时候，由于 PDF 的页码过多，所以计算出的高度也很大，而此时 PDFView 高度设置就会被警告

> -[<CALayer: 0x1d0227980> display]: Ignoring bogus layer size (375.000000, 408748.235294), contentsScale 3.000000, backing store size (1125.000000, 1226244.705882)

并且不显示该 UIView 图层，因此需要根据滑动来动态设置高度，并增加滚动绘制 PDF，类似分页的效果。还在实现中。

解决：应该是由于子页面的警告，这部分现在没解决，不过目前测试没遇到，得再测试测试。


#### PDFKit

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;它是苹果在 WWDC2017 发布的新 Kit，所以只支持 iOS11+。与 C语言 获取 PDF，提供的了简便的 API，相应的 API 大纲参考 《[PDFKit | Apple Developer Documentation](https://developer.apple.com/documentation/pdfkit?language=objc)》

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;其 PDFView + PDFDocument 就相当前一节开发的 [QHPDFView + CGPDFDocument](#QHPDFView + CGPDFDocument)。但 PDFKit 还提供了 PDFThumbnailView、及强大的 PDF 编辑（划线、涂鸦、水印等），还有增加 Page 等，并且 PDFDocument 可以获取更多的 PDF 信息（目录结构、页数、页码等）。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;加水印，可参考：《[Custom Graphics | Apple Developer Documentation](https://developer.apple.com/documentation/pdfkit/custom_graphics?language=objc)》

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;觉得苹果是发现前面提供封装好的 [QLPreviewController & UIDocumentInteractionController](#QLPreviewController & UIDocumentInteractionController) 使用率不高，自定义低，因此发布 PDFKit 让开发者可以组合使用，自由选择，且降低 PDF 常用信息解析的复杂度。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;如果 App 针对 iOS11+，完全可以使用它开发 PDF 阅读器。

#### PDF 解析

PDF 的读取对象就是 PDFDocument，从中可以读取 PDF 所有数据。

* CGPDFDocumentRef
* CGPDFDocument
* PDFDocument

##### CGPDFDocumentRef

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在《[提取pdf目录的方法](http://phaibin.tk/2012/01/06/ti-qu-pdfmu-lu-de-fang-fa)》的 Demo 里面就有如下代码，是使用 C语言 来获取 PDF 对象的信息。

```
static const void *
getCFDataBytePointer(void *info)
{
    CFDataRef data;
    
    data = info;
    return CFDataGetBytePtr(data);
}

static void
releaseCFData(void *info)
{
    CFDataRef data;
    
    data = info;
    CFRelease(data);
}

/* Create a direct-access data provider using `data', a CFDataRef. */

static CGDataProviderRef
dataProviderWithCFData(CFDataRef data)
{
    void *info;
    size_t size;
    static const CGDataProviderDirectCallbacks callbacks = {
        0,
        &getCFDataBytePointer, NULL, NULL, &releaseCFData
    };
    
    if (data == NULL)
        return NULL;
    
    size = CFDataGetLength(data);
    info = (void *)CFRetain(data);
    return CGDataProviderCreateDirect (info, size, &callbacks);
}

- (BOOL)loadDataRepresentation:(NSData *)data ofType:(NSString *)type
{
    CGDataProviderRef provider;
    
    if ([type isEqualToString:@"PDFType"]) {
        provider = dataProviderWithCFData((__bridge CFDataRef)data);
        if (data == NULL)
            return NO;
        document = CGPDFDocumentCreateWithProvider(provider);
        if (document == NULL) {
            CGDataProviderRelease(provider);
            return NO;
        }
        CGDataProviderRelease(provider);
        return YES;
    }
    
    return NO;
}
```

##### CGPDFDocument

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;其实 CGPDFDocumentRef 就是 CGPDFDocument，因此上述直接使用 CGPDFDocument 来获取将更加简便，代码如下

```swift
if let docu = document {
    if let page = docu.page(at: 1) {
        let rect = page.getBoxRect(.cropBox)
        let count = docu.numberOfPages
    }
}
```

##### PDFDocument

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;它是前面介绍的 PDFKit 里面提供的对象，拥有对 PDF 更丰富的读取、操作的 API。

```swift
guard let path = Bundle.main.path(forResource: "mp4", ofType: "pdf") else {
    return
}
let url = URL(fileURLWithPath: path)
pdfDocument = PDFDocument(url: url)
let count = pdfDocument.pageCount
pdfDocument.delegate = self
    
pdfView.document = pdfDocument
pdfView.displayMode = .singlePage
pdfView.autoScales = true
```

#### PDF 的结构

通过前面的工具可以查询 PDF 的 Metadata、Outlines（目录）等等大部分信息。

其二进制信息读取还是得参考 《[Adobe 的 PDF32000_2008.pdf](http://www.adobe.com/content/dam/Adobe/en/devnet/pdf/pdfs/PDF32000_2008.pdf)》

#### 参考：

1、 [歪樣 - 简书](https://www.jianshu.com/u/0c84bae43ced) 的如下文章中，能了解到 PDF 的具体操作。

* [iOS PDF文件预览的几种方法](https://www.jianshu.com/p/95168c23fb39)
* [iOS PDF文件格式转换](https://www.jianshu.com/p/b6fb4d662969)
* [iOS 从其他App获取文件](https://www.jianshu.com/p/978d38533c5c)
* [iOS PDF文档批注与修改](https://www.jianshu.com/p/888371cdb1b7)

2、PDFKit

* [使用PDFKit写一个基本的PDF阅读器](https://blog.csdn.net/cairo123/article/details/78615901)
* [iOS11 PDFKit 使用例程 - tzshlyt的个人空间](https://my.oschina.net/u/3691429/blog/1540042)

3、结构

* [提取pdf目录的方法](http://phaibin.tk/2012/01/06/ti-qu-pdfmu-lu-de-fang-fa)
* [一个简单PDF文件的结构分析](https://blog.csdn.net/adolphfend/article/details/24729025)

