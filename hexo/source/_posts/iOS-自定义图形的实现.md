---
title: iOS 自定义图形的实现
date: 2018-07-02 06:44:15
tags: 
  - Swift
  - layer
  - animation
categories:
  - iOS

---
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;自定义图形在产品的开发慢慢占据愈来愈多的时间，当然还有性能优化。这其实都是一般开发者特别想忽略或者很难长期坚持开发的内容。如这种自定义图形和动画，简单点不是又不耗性能，又方便开发，其实倒觉得是开发者都比较懒，能简单肯定更好。但懂的偷懒和不懂的偷懒天差地别了。动效这块还有很多知识点，特别是 FB 开源的 [pop](https://github.com/facebook/pop) 库。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;回到本次的需求需求，***倒计时 & 水波纹***，用在产品的连麦申请和连麦成功的状态。它们的需求说明分别为：倒计时，是个圆圈弧形进度条，中间就是倒计时的时间数字表示向用户发起连麦的显示，倒计时用于计算用户是否接受连麦请求。而水波纹，即水面的波纹一样，一圈一圈的扩散，用来表示当前用户正在与主播进行语音连麦。接下来会将代码直接全部贴出，可能有点多，是将项目的这部分代码移到 Playground 实现，也就是只需将代码拷贝到 Playground 就可以看到效果。但是需要注意，只有第一个 UIView 的倒计时看到效果（记得点开右边的显示按钮喔），而其它无法看到效果，可能得复制到 Demo ，也是 OK 的。

<!-- more -->

#### 自定义图形

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;绘制自定义图形，一般用 UIView ，大多情况是加圆角，矩形和圆形之类的。而当需要绘制同心圆，五角星这类比较复杂图形的时候，可能几个 UIView 也无法拼凑出来。因此就需要通过以下来实现。

##### UIView

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;通过它的 ***open func draw(_ rect: CGRect)*** 来绘制图形。如经常使用的控件 UILabel ，其文本的显示说到底也是通过绘制出来。

![QHCountdownView](https://anakinpublicspace-1253727175.cos.ap-chengdu.myqcloud.com/blog/QHCountdownView.png)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;来试试手，绘制倒计时的圆圈和倒计时数值，代码如下：

```swift
import UIKit
import PlaygroundSupport
import CoreGraphics

class QHCountdownView: UIView {
    
    var count: CGFloat = 0
    var nowCount: CGFloat = 0
    
    override init(frame: CGRect) {
        super.init(frame: frame)
        backgroundColor = UIColor.white
    }
    
    required init?(coder aDecoder: NSCoder) {
        super.init(coder: aDecoder)
    }
    
    override func draw(_ rect: CGRect) {
        if let context = UIGraphicsGetCurrentContext() {
			  // 绘制文本属性
            let textStyle = NSMutableParagraphStyle()
            textStyle.alignment = .center
            var textAttributes = [NSAttributedStringKey : Any]()
            textAttributes[NSAttributedStringKey.font] = UIFont.systemFont(ofSize: 22)
            textAttributes[NSAttributedStringKey.foregroundColor] = UIColor.orange
            textAttributes[NSAttributedStringKey.paragraphStyle] = textStyle

            let text = NSString(format: "%.0f", nowCount)

            let strSize = text.size(withAttributes: textAttributes)
			  // 文本绘制
            let centerP = CGPoint(x: (rect.size.width - strSize.width)/2, y: (rect.size.height - strSize.height)/2)
            text.draw(at: centerP, withAttributes: textAttributes)
			  // 圆弧角度设置
            context.setLineWidth(2)
            context.setStrokeColor(UIColor.orange.cgColor)
            let c = CGPoint(x: rect.origin.x + rect.size.width/2, y: rect.origin.y + rect.size.height/2)
            let end = CGFloat(Float(2)*Float.pi) * CGFloat((count - nowCount)/count) - CGFloat(Float.pi/Float(2))
            context.addArc(center: c, radius: rect.size.height/2-CGFloat(2), startAngle: CGFloat(-Float.pi/2), endAngle: end, clockwise: false)
            // 圆弧绘制
            context.drawPath(using: .stroke)
        }
    }
    // 再次刷新图形，通过比例计算其新的圆弧角度
    func progress(count: CGFloat) {
        nowCount = max(min(self.count, count), CGFloat(0))
        setNeedsDisplay()
    }
}

class MyViewController : UIViewController {
    
    var cdTime: CGFloat = 15
    let countdownV = QHCountdownView(frame: CGRect(x: 100, y: 100, width: 100, height: 100))
    var countdownTimer: Timer?
    
    override func viewDidLoad() {
        self.view.addSubview(countdownV)
        countdownV.count = cdTime
        countdownV.nowCount = cdTime
        countdownTimer = Timer.scheduledTimer(withTimeInterval: 0.1, repeats: true) {[weak self] (timer) in
            if var cdT = self?.cdTime {
                self?.countdownV.progress(count: cdT)
                if cdT < CGFloat(0) {
                    cdT = CGFloat(15)
                    self?.releaseTimer()
                }
                else {
                    self?.cdTime -= CGFloat(0.1)
                }
            }
        }
    }
    
    private func releaseTimer() {
        if let cT = countdownTimer {
            if cT.isValid {
                cT.invalidate()
            }
            countdownTimer = nil
        }
    }
}
// Present the view controller in the Live View window
PlaygroundPage.current.liveView = MyViewController()
```

执行刷新 UI 的代码：

| 函数 | 说明 |  
| --- | --- |
| setNeedsDisplay() 	| 这是为了缓冲一下，要刷新的view都放在一起刷新，避免浪费性能。它会触发 drawRect 绘制 |
| layoutIfNeeded() 		| 此方法立即执行 |
| setNeedsLayout()	 	| 不是立即执行而是在下一个drawing cycle update中一起更新 |

##### CAShapeLayer

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;使用 layer 来绘制自定义图形。其实，UIView 绘制也是对它的 layer 的绘制。所以使用它是底层，并且也是通过给 layer 动效的。UIBezierPath 是常用的路径绘制，而 PS 软件都会有这个画图工具。

```swift
// CAShapeLayer 图层
var clayer = CAShapeLayer()
clayer.strokeColor = UIColor.orange.cgColor
clayer.fillColor = UIColor.clear.cgColor
clayer.contentsScale = UIScreen.main.scale
clayer.borderWidth = 2
clayer.lineCap = kCALineCapRound
    
self.view.layer.addSublayer(clayer)
    
let point = self.view.center
let angle = CGFloat(Float.pi)/6
let startAngle: CGFloat = CGFloat(3/2 * Float.pi) - (CGFloat(Float.pi)/2 - angle)
let endAngle = CGFloat(Float.pi)-(CGFloat(Float.pi)/2) - (CGFloat(Float.pi)/2 - angle)
// UIBezierPath 路径，相当于前一部分的 context.addArc
let path = UIBezierPath(arcCenter: point, radius: 50, startAngle: startAngle, endAngle: endAngle, clockwise: false)
    
clayer.path = path.cgPath
```

#### 动画  

##### UIView  

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;UIView animation 具有显性动画，在 animations 的 block 里面对 frame 进行修改的话，它会在设置的时间里持续显示图形的变化过程。用它来做一些平移、缩放等动画。

```swift
 UIView.animate(withDuration: 1.8, animations: {
            }, completion: { (finished) in
            })
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;回到需求的水波纹，采用 CAShapeLayer & UIView animation 来实现，然后通过 Timer 控制连续添加的水波，并配合渐隐和放大的动效，从而模拟出了现实的水波纹效果。当然真实水波的扩散还有物理计算，如阻力等，这里暂且忽略，采用简单均速来实现。

```swift
import UIKit
import PlaygroundSupport

class QHWaterView : UIView {
    override init(frame: CGRect) {
        super.init(frame: frame)
        self.backgroundColor = UIColor.white
    }
    
    required init?(coder aDecoder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
    
    override func draw(_ rect: CGRect) {
        let radius: CGFloat = 26
        let startAngle: CGFloat = 0
        let point = self.center
        let endAngle = CGFloat(2 * Float.pi)
		 // 绘制圆圈
        let path = UIBezierPath(arcCenter: point, radius: radius, startAngle: startAngle, endAngle: endAngle, clockwise: true)

        let shapeLayer = CAShapeLayer()
        shapeLayer.path = path.cgPath
        shapeLayer.strokeColor = UIColor.orange.cgColor
        shapeLayer.fillColor = UIColor.clear.cgColor

        self.layer.addSublayer(shapeLayer)
    }
}

class MyViewController : UIViewController {
    
    let actionView = UIView(frame: CGRect(x: 100, y: 100, width: 100, height: 100))
    var countdownTimer: Timer?
    
    override func viewDidLoad() {
        addWaterAction()
    }
    
    private func removeViewAction() {
        if let cT = countdownTimer {
            if cT.isValid {
                cT.invalidate()
            }
            countdownTimer = nil
        }
    }
    
    private func addWaterAction() {
        removeViewAction()
        countdownTimer = Timer.scheduledTimer(withTimeInterval: 0.6, repeats: true) {[weak self] (timer) in
        	  // 定时添加水波
            let waterView = QHWaterView(frame: self!.actionView.bounds)
            self?.view.addSubview(waterView)
            // 模拟水波的渐隐扩散
            UIView.animate(withDuration: 1.8, animations: {
                waterView.transform = CGAffineTransform(scaleX: 2, y: 2)
                waterView.alpha = 0
            }, completion: { (finished) in
                waterView.removeFromSuperview()
            })
        }
        RunLoop.current.add(countdownTimer!, forMode: .commonModes)
    }
}
// Present the view controller in the Live View window
PlaygroundPage.current.liveView = MyViewController()
```

##### CAKeyframeAnimation

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;这里的 clayer 是倒计时里的 CAShapeLayer，然后通过多个 CAKeyframeAnimation 的组合动画，并设置的时间，添加到 clayer 上，这样动画会按照 path 来的绘制，从而呈现动画效果。

```swift
// 创建多个 CAKeyframeAnimation
let startAnimation = CAKeyframeAnimation(keyPath: "startAnimation")
startAnimation.values = [0, 1]
startAnimation.duration = 4
let endAnimation = CAKeyframeAnimation(keyPath: "endAnimation")
endAnimation.values = [1, 0]
endAnimation.duration = 4
endAnimation.beginTime = 4
 
// 组合 CAKeyframeAnimation   
let animationGroup = CAAnimationGroup()
animationGroup.animations = [startAnimation, endAnimation]
animationGroup.duration = 8
animationGroup.setValue("animationStep3", forKey: "animationName")
clayer.add(animationGroup, forKey: nil)
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;CAKeyframeAnimation 可以作为单个动画，也可以通过 CAAnimationGroup 将多个动画组合在一起。而如果不设置单个的开始时间，所添加的动画会并发执行。所以这里通过 `endAnimation.beginTime = 4` 设置开始时间，从而达到顺序执行的效果。

参考：  
- 《[放肆的使用UIBezierPath和CAShapeLayer画各种图形](https://www.jianshu.com/p/c5cbb5e05075)》  
- 《[干货系列之手把手教你使用Core animation 做动画](https://www.jianshu.com/p/1e2b8ff3519e)》  
- 《[iOS动画（Core Animation）总结](http://www.cocoachina.com/ios/20160712/17010.html)》  


#### 总结

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;通过以上的知识点，简单地了解和掌握了绘制自定义图片和动效。例子比较简单，但复杂的图形和动效，原理离不开上述内容，也就是再复杂也是通过简单组合而成，关键看代码如果编写。程序员其实也是个画家，也是个动画片制作者，只是使用的是代码。