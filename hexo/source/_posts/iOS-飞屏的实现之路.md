---
title: iOS 飞屏的实现之路
date: 2018-06-26 11:58:58
tags:
  - layer
  - 动画
  - GCD
categories:
  - iOS
  
---
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;开发 **飞屏** 功能，跟我较早前的弹幕系统相似 [QHDanumuDemo: 弹幕系统](https://github.com/chenqihui/QHDanumuDemo)。不同之处在于，弹幕系统需要计算弹幕轨道的空闲与否，弹幕是否处在相互碰撞位置，时间校准等控制每条弹幕的出现，并且大多数弹幕系统是有多种位置（上中下），及不同出现方式，最近 bilibili 实现了弹幕绕开人脸的实现，cool。那么飞屏呢，它一般是固定规律显示的条数，并且动画较少，但是飞屏的样式比弹幕复杂（我目前的需求是这样，这个不是重点）。抛开样式，从逻辑来讲，飞屏其实就是简化版的弹幕系统。本次需求由于要集成 SDK，所以不打算在原有的弹幕系统增加支持。以重新开发，去掉复杂逻辑，也就抽出今天的内容。

<!-- more -->

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;需求说明：心愿飞屏，在直播间通过飞屏的形式展示用户的心愿。通过接收 Socket 数据即显示，Socket 数据是一条一条下发（这样做是可以由服务端来飞屏的控制间隔时间，当然也可以批量接收），然后创建飞屏 UI，开始从右向左飞出。而飞屏上可以点赞某条心愿。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;飞屏的实现效果截图如下：

![飞屏GIF](http://pacfu36li.bkt.clouddn.com/QHFlyView.gif)

#### CALayer

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;它是 iOS 实现自定义图形及动画的必要之路，这里可看后续的篇章[TODO]

#### Animation

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在现在的 App 对于使用者看起来越来越复杂，可能动画居多。早期（或者是 App 开始之初）产品提出的需求，大多是静态的 View（没有动效的概念，大多是程序员自己想象，或者使用系统的），而后面有交互设计师后，开始加入各种特效、如动画、音效等。对于动画，常用的有 UIImageView 的序列帧播放多张 UIImage，或者 WebP 及 GIF，这些主要是依靠设计制作的图片集。对于开发人员，需要自己开发的动效，常用的是 UIView animate 这种动画，它也能为飞屏提供从右向左的飞行效果。

但是本次实现这部分动效，主要使用了 CABasicAnimation & CAKeyframeAnimation，它们是另一套苹果提供的动画 API，可以调节更多序列帧动画细节的参数。

《[ObjC 中国 - View-Layer 协作](https://objccn.io/issue-12-4/)》该文介绍了动画实现及理论。

##### 飞行的动画

```objc
- (void)p_startAnimationCoins:(UIView<CAAnimationDelegate> *)subView duration:(CGFloat)timeDuration {
    CABasicAnimation *animation = [CABasicAnimation animationWithKeyPath:@"position"];
    animation.fromValue =  [NSValue valueWithCGPoint:subView.layer.position];
    CGPoint toPoint = subView.layer.position;
    toPoint.x = -subView.frame.size.width;
    animation.toValue = [NSValue valueWithCGPoint:toPoint];
    animation.duration = timeDuration;
    animation.fillMode = kCAFillModeForwards;
    animation.removedOnCompletion = NO;
    animation.delegate = subView;
    [subView.layer addAnimation:animation forKey:@"position"];
}
```
##### 点赞的动画

```objc
- (void)clickHeart {
    if (self.heartIV.isHighlighted == NO) {
        [self.heartIV setHighlighted:YES];
        
        CGFloat duration = 0.6;
        CAKeyframeAnimation *popAnimation = [CAKeyframeAnimation animationWithKeyPath:@"transform"];
        popAnimation.duration = duration;
        popAnimation.values = @[[NSValue valueWithCATransform3D:CATransform3DIdentity],
                                [NSValue valueWithCATransform3D:CATransform3DMakeScale(0.3f, 0.3f, 1.0f)],
                                [NSValue valueWithCATransform3D:CATransform3DIdentity]];
        popAnimation.keyTimes = @[@.1, @0.5f, @1.0f];
        popAnimation.timingFunctions = @[[CAMediaTimingFunction functionWithName:kCAMediaTimingFunctionEaseIn], [CAMediaTimingFunction functionWithName:kCAMediaTimingFunctionEaseOut]];
        [self.heartIV.layer addAnimation:popAnimation forKey:nil];
    }
}
```

#### TapGesture

##### v1

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;飞屏的点赞。一般这种点击实现，通过添加 UIButton 或 UIControl 来实现，不过由于飞屏是飞行的，且多条，跟弹幕系统一样。所以需要为整个区域添加一个 TapGesture 手势，通过手势点击响应得到的 location 来获取对应的飞屏 view。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;这里需要通过呈现图层 self.layer.presentationLayer 来获取。

《[CALayer之presentationLayer和ModelLayer](https://www.jianshu.com/p/d50b0e13905c)》
>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;每个图层属性的显示值都被存储在一个叫做呈现图层的独立图层当中，他可以通过-presentationLayer方法来访问。这个呈现图层实际上是模型图层的复制，但是它的属性值代表了在任何指定时刻当前外观效果。换句话说，你可以通过呈现图层的值来获取当前屏幕上真正显示出来的值。

```objc
- (void)tapClickAction:(UITapGestureRecognizer *)tap {
    CALayer *tappedLayer;
    CGPoint touchPoint = [tap locationInView:self];
    touchPoint = CGPointMake(touchPoint.x + self.frame.origin.x, touchPoint.y + self.frame.origin.y);
    tappedLayer = [self.layer.presentationLayer hitTest:touchPoint];
    id layerDelegate = [tappedLayer.superlayer delegate];
    if ([layerDelegate isKindOfClass:[CellView class]]) {
        CellView *cellView = (CellView *)layerDelegate;
        [cellView clickHeart];
    }
}
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;需要注意 ***locationInView*** 是获取相对位置。因为这里用的是父类的 presentationLayer ，所以 ***[self.layer.presentationLayer hitTest:touchPoint]*** 是以整个屏幕，需要再计算对应的 touchPoint 才能得到需要的飞屏上的 layer。***tappedLayer.superlayer*** 这里是获取飞屏的根 layer。之前发布的 [QHGoldCoinsMan：抢金币](https://github.com/chenqihui/QHGoldCoinsMan) 和 弹幕系统 都没留意，因为它们的 cell 只是一个控件（UIImageView 或者 UILabel），所以获取的 tappedLayer 就是根  layer。可是这里飞屏布局控件样式不同，它由多个控件构成，因此 tappedLayer 只能拿到最上面的 layer，不是飞屏的根 layer，需要通过 superlayer 来获取（类似 superclass 或 superview）。可是又无法确切知道需要 superlayer 获取多少层。最后解决方案是在飞屏最后加上 maskView，也就是说点击获取的 tappedLayer 是这个 maskView，再通过一次 superlayer 即获取到根 layer（如果有其他方案可以告知）。

```objc
UIView *maskV = [[UIView alloc] init];
maskV.backgroundColor = [UIColor clearColor];
[cellView addSubview:maskV];
```

##### v2

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;v1 的时候讲了使用 maskView，再通过一次 superlayer 即获取到根 layer，实践是OK。但是当这个飞屏的整个区域还存在的时候，其手势就会阻碍底部的其他点击事件，这样就得在没有飞屏的时候回收它。开始是先通过 NSTimer 来控制，即超出时间没有 addCell 的时候就 remove view。但还是觉得一般，最终决定通过将手势添加到每个飞屏上（当然大伙可以自己评估下孰优孰劣）。

```objc
UITapGestureRecognizer *tap = [[UITapGestureRecognizer alloc] initWithTarget:cellView action:@selector(tapClickAction:)];
[cellView addGestureRecognizer:tap];
``` 

通过 hitTest 这个飞屏的 self.layer.presentationLayer 来判断是否点击的是飞行区域，呈现图层才是飞屏在飞行时的位置。

```objc
- (void)tapClickAction:(UITapGestureRecognizer *)tap {
    CGPoint touchPoint = [tap locationInView:self.superview];
    if ([self.layer.presentationLayer hitTest:touchPoint]) {
        [self clickHeart];
    }
}
```

然后，父 view 添加 hitTest 响应函数来过滤自己，这样就只响应子 view，即飞屏。

```objc
- (nullable UIView *)hitTest:(CGPoint)point withEvent:(nullable UIEvent *)event {
    UIView* tmpView = [super hitTest:point withEvent:event];
    
    if (tmpView == self) {
        return nil;
    }
    return tmpView;
}
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;通过上述逻辑，就不用再考虑飞屏区域问题（即不用移除它），因为只在有飞屏的时候，其才会响应点击。

#### 线程

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;这一部分主要想复习下《Objective-C高级编程 iOS与OS X多线程和内存管理》里面关于 GCD 的介绍。而使用 GCD 的同步就是为了避免数据的读写问题，因为在 insert -> show -> remove 都是操作同一个数据源，按理可以添加同步锁。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;先聊下飞屏的测试，简单地通过循环 insert 数据，然后展示出飞屏。最初基数为 100、2000 的时候并没有发现什么问题，即飞屏正常飞行。(以下代码使用 Playground 来实现，效果可能有出入)

```swift
let count = 100
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;而当将 count 调整到 20000 或者更高的时候，App 直接卡住了。

```swift
print("start")
for _ in 1..<count {
    DispatchQueue.main.async {
        print("insert")
    }
}
print("end")
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;发现原来是 GCD 造成的，因为这里多次调用 DispatchQueue.main.async。修改就很简单，只有把 GCD 代码移到 for 循环外面，其实本来就得这么做。

```swift
DispatchQueue.main.async {
    print("start")
    for _ in 1..<count {
        print("insert")
    }
    print("end")
}
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;接着再想想，这里是为了确保 insert 飞屏的数据才放在主线程。而 insert 是将要提供给外部调用的 API。那么就必须要求调用者在主线程调用才能确保正常，这有点类似 UI 的操作，好像也合理。可是这样是不是有点麻烦！不如修改成如下

```swift
print("start")
for _ in 1..<count {
    if Thread.isMainThread == true {
        print("insert")
    }
    else {
        DispatchQueue.main.async {
            print("insert")
        }
    }
}
print("end")
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;以上操作可能在 Playground 不会卡住，可是 App 真实项目却把 Xcode 卡挂。 Playground 运行主线程的操作需要加一些声明

《[DispatchQueue.main now working |Apple Developer Forums](https://forums.developer.apple.com/thread/8223)》

```swift
import PlaygroundSupport
PlaygroundPage.current.needsIndefiniteExecution = true
```

