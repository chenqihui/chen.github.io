---
title: iOS 表单开发 基础篇
date: 2018-10-06 18:55:58
tags:
  - Swift
  - UITextField
  - UITextView
  - UIButton
categories:
  - iOS
  - 基础
  
---
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;移动端大部分页面是由列表、表单、对话框展示，它们也是开发中最频繁开发的部分。说说表单，它是提供用户输入数据后提交的页面，常用于注册、登录、购物、评论等等。将表单抛开复杂控件和动画等元素，其主要由输入框、选择框、按钮构成，逻辑也就是输入、选择要提交的内容，点击提交即完成，这只是简化的逻辑。而完整的表单，其实可能根据业务不同，会有不同的细节处理，如加密，数据校验，UI 联调，状态提醒等等。那接下来介绍几个比较常见的表单设置与开发。

<!-- more -->

#### 输入框 

##### UITextField & UITextView

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;分别是单行、多行文本输入与展示的控件

```swift
let testTF = UITextField(frame: CGRect(x: 0, y: 0, width: 100, height: 30))
```

##### 输入的键盘类型

```swift
testTF.keyboardType = .default

public enum UIKeyboardType : Int {

    
    case `default` // Default type for the current input method.

    case asciiCapable // Displays a keyboard which can enter ASCII characters

    case numbersAndPunctuation // Numbers and assorted punctuation.

    case URL // A type optimized for URL entry (shows . / .com prominently).

    case numberPad // A number pad with locale-appropriate digits (0-9, ۰-۹, ०-९, etc.). Suitable for PIN entry.

    case phonePad // A phone pad (1-9, *, 0, #, with letters under the numbers).

    case namePhonePad // A type optimized for entering a person's name or phone number.

    case emailAddress // A type optimized for multiple email address entry (shows space @ . prominently).

    @available(iOS 4.1, *)
    case decimalPad // A number pad with a decimal point.

    @available(iOS 5.0, *)
    case twitter // A type optimized for twitter text entry (easy access to @ #)

    @available(iOS 7.0, *)
    case webSearch // A default keyboard type with URL-oriented addition (shows space . prominently).

    @available(iOS 10.0, *)
    case asciiCapableNumberPad // A number pad (0-9) that will always be ASCII digits.

    
    public static var alphabet: UIKeyboardType { get } // Deprecated
}
```

##### 输入结束的键盘回收

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;UITextFieldDelegate

```
//MARK: UITextFieldDelegate
    
public func textFieldShouldReturn(_ textField: UITextField) -> Bool {
    textField.resignFirstResponder()
    return false
}
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在使用键盘回收的时候，可能会有输入面板的升降等类似 UI 的变化。当使用原生的键盘时候都是点击 return 作为确定按钮回收键盘。可是当使用第三方键盘时候，有多一个工具区域（即在正常键盘上方），其会有回收按钮，它不会触发上述的回调函数。因此需要增加如下回调

```
- (void)textFieldDidEndEditing:(UITextField *)textField {
	// doAction
}
```

#### 键盘

##### 键盘监听

```
import UIKit

class QHKeyboardEvent: NSObject {
    
    class func removeKeyboardNotificationCenter(object: Any) {
        NotificationCenter.default.removeObserver(object, name: NSNotification.Name.UIKeyboardWillShow, object: nil)
        NotificationCenter.default.removeObserver(object, name: NSNotification.Name.UIKeyboardWillHide, object: nil)
    }
    
    class func addKeyboardNotificationCenter(object: Any, keyboardWillShow willShow: Selector, keyboardWillHide willHide: Selector) {
        NotificationCenter.default.addObserver(object, selector: willShow, name: NSNotification.Name.UIKeyboardWillShow, object: nil)
        NotificationCenter.default.addObserver(object, selector: willHide, name: NSNotification.Name.UIKeyboardWillHide, object: nil)
    }
    
    class func keyboardActionWithNotification(_ notification: Notification, complete: (_ hKeyB: CGFloat) -> Void) {
        if let userInfo = notification.userInfo {
            let animationCurveObject = userInfo[UIKeyboardAnimationCurveUserInfoKey] as! NSValue
            let animationDurationObject = userInfo[UIKeyboardAnimationDurationUserInfoKey] as! NSValue
            let keyboardEndRectObject = userInfo[UIKeyboardFrameEndUserInfoKey] as! NSValue
            
            var animationCurve = UIViewAnimationCurve.easeOut
            var animationDuration = 0.0
            var keyboardEndRect = CGRect(x: 0, y: 0, width: 0, height: 0)
            animationCurveObject.getValue(&animationCurve)
            animationDurationObject.getValue(&animationDuration)
            keyboardEndRectObject.getValue(&keyboardEndRect)
            
            var hKeyB = 0 as CGFloat
            hKeyB = keyboardEndRect.size.height
            
            UIView.beginAnimations("qhKeyboardAction", context: nil)
            UIView.setAnimationDuration(animationDuration)
            UIView.setAnimationCurve(animationCurve)
            complete(hKeyB)
            UIView.commitAnimations()
        }
    }
}
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;使用

```
func p_addNotification() {
    QHKeyboardEvent.addKeyboardNotificationCenter(object: self, keyboardWillShow: #selector(self.keyboardWillShow(notification:)), keyboardWillHide: #selector(self.keyboardWillHide(notification:)))
}
    
func p_removeNotification() {
    QHKeyboardEvent.removeKeyboardNotificationCenter(object: self)
}
```

##### 键盘弹出或者回收时，操作输入面板的上升与下降

```
1、使用约束
@IBOutlet weak var contentVTopLC: NSLayoutConstraint!
2、frame 进行修改
CGRect frame
```

#### 按钮

##### 单色状态

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;使用 qhImageWithColor & setBackgroundImage 来设置按钮各种的单色状态： normal disable highlight：

```
// 正常状态的颜色
doBtn.setBackgroundImage(SHRewardCashViewController.qhImageWithColor(UIColor(hexValue: 0x189AFF)), for: UIControlState.normal)
        
// 禁止点击状态的颜色doBtn.setBackgroundImage(SHRewardCashViewController.qhImageWithColor(UIColor(hexValue: 0x189AFF, alpha: 0.5)), for: UIControlState.disabled)
```

```
class func qhImageWithColor(_ color: UIColor) -> UIImage? {
    let rect = CGRect(x: 0, y: 0, width: 100, height: 100)
    UIGraphicsBeginImageContext(rect.size)
    let context = UIGraphicsGetCurrentContext()
    context?.setFillColor(color.cgColor)
    context?.fill(rect)
    
    let image = UIGraphicsGetImageFromCurrentImageContext()
    UIGraphicsEndImageContext()
    return image
}
```

##### 圆角

```
// 圆角
doBtn.layer.masksToBounds = true
doBtn.layer.cornerRadius = 10
// 边距
doBtn.layer.borderWidth = 1
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;而使用 storyboard 或者 xib 的，可以在属性面板 User Defined Runtime Attributes 里添加，如下：

| Key Path | Type | Value |
| --- | --- | --- |
| layer.masksToBounds | Boolean | √ |
| layer.cornerRadius | Number | 10 |

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;但是 cornerRadius 是需要计算的，就只能使用代码进行设置。

##### 数据校验

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在提交数据的时候需要对数据的校验

1、前置校验限制

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;通过 UITextFieldDelegate 的限制输入

```
public func textField(_ textField: UITextField, shouldChangeCharactersIn range: NSRange, replacementString string: String) -> Bool {
    if textField == accountTF {
        if let phoneNumber = textField.text {
            if phoneNumber.count >= 11 && !string.isEmpty {
                return false
            }
        }
    }
    return true
}
```

2、对于满足条件才允许点击提交按钮

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;一般通过 UIControlEvents.editingChanged 状态通知获得输入变化的实时校验控制

``` 
accountTF.addTarget(self, action: #selector(self.updateLoginButtonAction(sender:)), for: UIControlEvents.editingChanged)

func p_updateLoginButton() {
    if accountTF.text?.isEmpty == true || passwordTF.text?.isEmpty == true {
        sureButton.isEnabled = false
    }
    else if p_isHiddenPictureCode() == false && pictureTF.text?.isEmpty == true {
        sureButton.isEnabled = false
    }
    else {
        sureButton.isEnabled = true
    }
}

@objc func updateLoginButtonAction(sender: UITextField) {
    p_updateLoginButton()
}
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;多个输入联动按钮 UIButton 是否点击，即给多个 UITextField 添加监听来控制

##### RxSwift

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;由于上述的方法都是使用系统的回调与通知来控制，而越来越大第三方库提供更便捷的方法，如 RxSwift

1、单个输入

```
cashTF.rx.text.orEmpty.changed
            .subscribe(onNext: {
                if let b = Float($0) {
                    if b > self.balance {
                        self.errorTipL.isHidden = false
                        return
                    }
                }
                self.errorTipL.isHidden = true
            })
            .disposed(by: disposeBag)
```

2、多个联动

```
Observable.combineLatest(cashTF.rx.text.orEmpty, nameL.rx.text.orEmpty, accountL.rx.text.orEmpty) { (cash, name, account) -> Bool in
                if cash.count > 0, name.count > 0, account.count > 0, self.errorTipL.isHidden == true {
                    return true
                }
                return false
            }
            .map {
                self.doBtn.isEnabled = $0
            }
            .bind {}
            .disposed(by: disposeBag)
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;这篇不对 RxSwift 进行扩展介绍

#### 进阶[TODO]

校验内容：手机号码、邮箱、身份证

根据填入数据显示其他输入控件，如验证码等

图片、头像、视频

提交后数据更新

对话框形式、动画、居中布局