---
title: iOS 核心动画高级技巧
date: 2018-08-23 15:04:50
tags:
  - layer
  - 动画
categories:
  - iOS
  
---
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;如果阅读前面的文章 《[iOS 自定义图形的实现 | RyuukuSpace](http://chenqihui.github.io/2018/07/02/iOS-%E8%87%AA%E5%AE%9A%E4%B9%89%E5%9B%BE%E5%BD%A2%E7%9A%84%E5%AE%9E%E7%8E%B0/)》 和 《[PDF 预览 & 解析 | RyuukuSpace](http://chenqihui.github.io/2018/08/16/PDF-%E9%A2%84%E8%A7%88-%E8%A7%A3%E6%9E%90/)》，会发现里面就使用了 CAShapeLayer 和 CATiledLayer。其实它们是苹果提供视图更底层的操作——图层，对于 UIView 大多数人都比较了解和常用，UIView 其实就是在 layer 的基础上加工，并提供更加高级的操作，而有些时候还是需要使用 layer 来进行更高效，更专有的使用。接下来就推荐下关于 layer 的一本翻译书籍。

<!-- more -->

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[chenqihui/iOS-Core-Animation-Advanced-Techniques: 翻译](https://github.com/chenqihui/iOS-Core-Animation-Advanced-Techniques) 这个仓库，印象很早的时候就 fork 下来，那时候也看了一遍，不过现在可以说是忘干净了。最近由是于项目开发使用 layer，查找资料时候有文章是转载其中的译文，才想起再次翻阅它。而它是 Github 上的一个翻译版，主要介绍了 layer 的相关知识点，可以说是非常详细，也翻译得很好，并且最近发现更新之后，还加入了 Gitbook 版，真的相当给力。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Gitbook 版：《[iOS 核心动画高级技巧](http://zsisme.gitbooks.io/ios-/)》**

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;由于内容较多就不再重复转载，采用链接方式。可以选择阅读 Gitbook 整本，也可以通过下面目录单独阅读对应的 md 译文，任君选择。

内容目录：
----

翻译，喵~

>知识是人类进步的阶梯

1. [图层树](https://github.com/chenqihui/iOS-Core-Animation-Advanced-Techniques/blob/master/1-图层树/图层树.md)
2. [寄宿图](https://github.com/chenqihui/iOS-Core-Animation-Advanced-Techniques/blob/master/2-寄宿图/寄宿图.md)
3. [图层几何学](https://github.com/chenqihui/iOS-Core-Animation-Advanced-Techniques/blob/master/3-图层几何学/图层几何学.md)
4. [视觉效果](https://github.com/chenqihui/iOS-Core-Animation-Advanced-Techniques/blob/master/4-视觉效果/4-视觉效果.md)
5. [变换](https://github.com/chenqihui/iOS-Core-Animation-Advanced-Techniques/blob/master/5-变换/变换.md)
6. [专有图层](https://github.com/chenqihui/iOS-Core-Animation-Advanced-Techniques/blob/master/6-专有图层/6-专有图层.md)
7. [隐式动画](https://github.com/chenqihui/iOS-Core-Animation-Advanced-Techniques/blob/master/7-隐式动画/隐式动画.md)
8. [显式动画](https://github.com/chenqihui/iOS-Core-Animation-Advanced-Techniques/blob/master/8-显式动画/显式动画.md)
9. [图层时间](https://github.com/chenqihui/iOS-Core-Animation-Advanced-Techniques/blob/master/9-图层时间/图层时间.md)
10. [缓冲](https://github.com/chenqihui/iOS-Core-Animation-Advanced-Techniques/blob/master/10-缓冲/缓冲.md)
11. [基于定时器的动画](https://github.com/chenqihui/iOS-Core-Animation-Advanced-Techniques/blob/master/11-基于定时器的动画/基于定时器的动画.md)
12. [性能调优](https://github.com/chenqihui/iOS-Core-Animation-Advanced-Techniques/blob/master/12-性能调优/性能调优.md)
13. [高效绘图](https://github.com/chenqihui/iOS-Core-Animation-Advanced-Techniques/blob/master/13-高效绘图/13-高效绘图.md)
14. [图像IO](https://github.com/chenqihui/iOS-Core-Animation-Advanced-Techniques/blob/master/14-图像IO/图像IO.md)
15. [图层性能](https://github.com/chenqihui/iOS-Core-Animation-Advanced-Techniques/blob/master/15-图层性能/15-图层性能.md)