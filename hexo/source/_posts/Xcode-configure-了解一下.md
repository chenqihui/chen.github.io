---
title: Xcode configure 了解一下
date: 2018-11-05 10:59:53
tags:
  - Swift
  - configure
categories:
  - Xcode
  - 基础
  
---
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;对于接收其他团队的 App 项目的时候，需要了解其中一项内容就是它的多环境，如测试、开发、发布等不同环境的逻辑。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;对于多环境的使用，之前本人使用较多的是配置多个 target，然后在 不同target 里面配置不同的条件宏。targets 可以是 duplicate target，也可以是新建 Application、Extension 或者 Framework 等等，常用的 通知扩展、手表扩展等

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;而此次接手的项目使用的是 .xcconfig 文件。那么就来大概了解下它的文件、配置 & 使用。

<!-- more -->


#### 1、文件

##### 1.1、workspace -> project -> targets

>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;workspace，顾名思义就是我们的工作区。一个workspace可以包含多个project以及一些其它文件。workspace也可以把多可以project组织起来。
>
>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;一个project会包含属于这个项目的>所有文件，资源，以及生成一个或者多个软件产品的信息。
>
>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;一个project会包含一个或者多个 target，而每一个 target都对应一个products，也就是最终产生的.app。
>
>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;一个targets可以有多个configuration（如我们平常用到的debug和release，当然我们还可以自己添加），每个configuration就会有对应的build settings。每次build都是在一个configuration下build的。
>
>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;build setting 中包含了 product 生成过程中所需的参数信息。project的build settings会对于整个project 中的所有targets生效，而target的build settings是重写了project的build settings，重写的配置以target为准。
>
>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;那么，什么又是scheme呢？scheme就相当于一个组织者。在build的时候，schema会指定一个target和configuration，这样就能保证在build的时候configuration的唯一性，就能产生一个特定的product。

![0](http://pacfu36li.bkt.clouddn.com/xcconfig0.png)

参考：

[xcconfig 的使用与 xcode 环境变量](https://www.jianshu.com/p/aad1f9e72382)

##### 1.2、了解 .xcconfig 文件、及 Project 设置

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;有个总体了解之后，可以再深入一点了解下 Target 和 Project。了解了 Target 和 Project 里面的配置关系后，就动手配置 .xcconfig 文件

参考：

[Xcode 使用 xcconfig文件 配置环境](https://www.jianshu.com/p/270e2c555d35)

#### 2、配置

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.xcconfig 文件 其实是一个键值对文件

内容一般如下

```
XCCONFIG_ACCOUNT = MOBILE

XCCONFIG_DISPLAY_NAME_MOBILE = MOBILE
XCCONFIG_DISPLAY_NAME_APPSTORE = APPSTORE

ENABLE_BITCODE = NO

XCCONFIG_DISPLAY_NAME = $(XCCONFIG_DISPLAY_NAME_$(XCCONFIG_ACCOUNT))

CONFIG_FLAG = FREE_VERSION_TEST //DEBUG

CONFIG_TEST = $(XCCONFIG_DISPLAY_NAME)
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;然后在 project->info 里面给 target 配置 configure

![0](http://pacfu36li.bkt.clouddn.com/xcconfig1.png)

#### 3、使用

##### 3.1、添加到项目

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;${XXXX}

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;可配置的使用的地方很多，如 项目名称、版本、URL Scheme 等。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;如针对项目 App 名称的设置，使用 ${XCCONFIG_DISPLAY_NAME} 后，它将根据 configure 文件的配置来填充内容。

![0](http://pacfu36li.bkt.clouddn.com/xcconfig2.png)

##### 3.2、添加到 info.plist

```
if let dictionary = Bundle.main.infoDictionary {
    if let appName = dictionary["Test"] as? String {
        print("appName: \(appName)")
    }
}
```

参考：

* [Xcode:用于管理多个 target 配置的 XCConfig 文件 - SwiftGG翻译组](https://segmentfault.com/a/1190000004080030)

##### 3.3、添加到 Swift 条件编译

![0](http://pacfu36li.bkt.clouddn.com/xcconfig3.png)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;（注意： Objective-C & Swift 项目的配置条件宏位置不同）

```
#if FREE_VERSION_TEST2
    print("app FREE_VERSION_TEST2")
#elseif FREE_VERSION_TEST
    print("app FREE_VERSION_TEST")
#else
    print("app FREE_VERSION3")
#endif
```

参考：

* [用xcconfig文件配置iOS app环境变量](http://www.cocoachina.com/ios/20160815/17360.html)
* [Swifter - Swift 必备 tips](https://swifter.tips/condition-compile/)

#### 4、总结

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;使用 configure 的优势在于文本话切统一管理，但是目前使用需要修改 对应的配置值，并且更新使用前需要 clean 之后才生效。如果使用分支管理，在发布分支已经修改好，也能避免重复修改。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;而 多个target 的配置是可以在 Xcode 或者 xcrun 选择，总的来说都能达到相同效果。



