---
title: iOS 动态库 应用篇
date: 2019-08-18 21:25:07
tags:
  - Framework
  - 动态库
categories:
  - iOS
  
---

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;动态库（Dynamic Framework），使用它的优点之一就是避免冲突（还有其他优点，如共享）。Xcode 的项目如果出现同名的文件和类在编译的时候就会报错。开发SDK 就需要非常注意引入第三方库等可能给其他团队出现同名的冲突情况。除了改名的方案之外，动态库也是一个好方案。本篇就将对最近应用动态库的过程进行记录 & 供参考。

<!-- more -->

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Framework 分为 Dynamic & Static 两种。可以在如下进行配置

~~~
Build Settings -> Linking -> Mach-O Type
~~~

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;查看现有的 Framework 是不是动态库，可以通过到 Framework 目录下

~~~
cd /QHFMWKLib.framework
file QHFMWKLib
# 输出：
# 静态库
QHFMWKLib: current ar archive random library
# 动态库
QHFMWKLib: Mach-O universal binary with 4 architectures: [i386:Mach-O dynamically linked shared library i386] [arm64:Mach-O 64-bit dynamically linked shared library arm64]
QHFMWKLib (for architecture i386):	Mach-O dynamically linked shared library i386
QHFMWKLib (for architecture x86_64):	Mach-O 64-bit dynamically linked shared library x86_64
QHFMWKLib (for architecture armv7):	Mach-O dynamically linked shared library arm_v7
QHFMWKLib (for architecture arm64):	Mach-O 64-bit dynamically linked shared library arm64
~~~

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;而本篇主要介绍的是 Dynamic Framework，所以文中其余部分写的 Framework 都是指的动态库， 除非特别强调是静态库。

* [Dynamic Library Programming Topics](https://developer.apple.com/library/archive/documentation/DeveloperTools/Conceptual/DynamicLibraries/000-Introduction/Introduction.html)


## 前言

背景：  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在开发 SDK 时候，使用了一个购买的第三方库支持的时候，由于有另一个团队 SDK（简称： B 团队） 也需要使用。为解决共用问题进行讨论，建议让 集成两个 SDK 的 App 团队（简称：A 团队）来管理此第三方库。但是 B 团队 却认为在如果第三方库升级上可能造成问题，坚决各自使用。所以才有了下面使用 动态库 来封装 .a 库。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;首先，分别创建如下三个工程：

| 工程名 | 生成的库 | 说明 |  
| --- | --- | --- |  
| QHALib.xcodeproj | QHALib.a & (*.h) | 静态库 |  
| QHFMWKLib.xcodeproj | QHFMWKLib.framework | 动态库 |  
| QHUseFMWKDemo.xcodeproj | QHUseFMWKDemo.app | Demo |


## 动态库的使用：

### 引入

~~~
General -> Embedded Binaries -> + Framework
~~~

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Xcode 会自动在 Linked Frameworks and Libraries 添加。这也是区别 静态 Framework 和 .a 库，因为它们只需添加 Linked Frameworks and Librarie 或者 Build Phase -> Link Binary With Libraries。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;注意：如果是通过拉拽到 Xcode 工程，Xcode 只会按静态库的方式配置，即不会添加到第一项，此时需要手动添加，不然编译时会报如下错误：

~~~
dyld: Library not loaded: @rpath/QHFMWKLib.framework/QHFMWKLib
  Referenced from: /Users/.../QHUseFMWKDemo.app/QHUseFMWKDemo
  Reason: image not found
~~~

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;如果使用 Cocoapods 集成的话，就无需考虑以上的情况。这是由于 podfile 里加入了如下将 pod install 生成的库由 Framework 替换 .a（iOS8 以上的默认支持）

~~~
Objective-C 项目：
#use_frameworks!
Swift 项目：
use_frameworks!
~~~

* [Pod Authors Guide to CocoaPods Frameworks](http://blog.cocoapods.org/Pod-Authors-Guide-to-CocoaPods-Frameworks/)

### 调用

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;引入成功后，只需要 import Framework/对应的头文件即可。如果遇到如下警告：

~~~
#import <QHFMWKLib/QHHelloGG.h> // Missing submodule 'QHFMWKLib.QHHelloGG'
~~~

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;是由于 Framework 除了需要将对外的 Header 放置在 Public 外，还需要在创建 Framework 时的同名头文件里，将这些 public 的头文件 import 一下，这样就能消除这种警告。


## 引入静态库的动态库

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;将 .a 库 引入 动态库 Framework（其实就是用动态库封装 .a 库），然后再给 Demo 使用。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;把 .a 库 和 头文件引入到 Framework 之后编译可能是 OK 的。可是当编译 Demo（引用的 .a 库的头文件时），可能会遇到 Undefined symbol（未定义符号）的情况

~~~
Undefined symbols for architecture x86_64:
  "_OBJC_CLASS_$_QHALib", referenced from:
      objc-class-ref in ViewController.o
ld: symbol(s) not found for architecture x86_64
clang: error: linker command failed with exit code 1 (use -v to see invocation)

Showing All Messages
:-1: Undefined symbol: _OBJC_CLASS_$_QHALib
~~~

### Undefined symbol

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;这是说明 Demo 在 Framework 的符号表里面找不到 .a 的类及函数。通过 **nm** 来查看 Framework 的符号表，可以发现确实不包含 .a 库的任何符合表。网上的部分解释：

>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;OC 的链接器并不会为每个方法建立符号表，而是仅仅为类建立了符号表。这样的话，如果静态库中定义了已存在的一个类的分类，链接器就会以为这个类已经存在，不会把分类和核心类的代码合起来。这样的话，在最后的可执行文件中，就会缺少分类里的代码，这样函数调用就失败了。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;那么如何解决呢？有两个办法：

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;第一种是通过 Framework 的类进行调用 .a 库的，通过封装的方式给外部调用，即不把 .a 的类暴露出去，使用这种间接的办法回避。如果调用的接口相对简单，或者是业务变动不大，这或许是一个不错的办法。而第二是方法是：Other Linker Flags

* [iOS framework/静态库 nm 调试](https://blog.csdn.net/u012703795/article/details/51490315)
* [Mac系统下lipo, ar, nm等工具的使用简介](https://www.jianshu.com/p/e0ef5146bd7a)

### Other Linker Flags

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;那通过在 Other Linker Flags 设置 **-all_load** 或者 **-force_load +（.a 库的 Path）**，如下

~~~
Build Settings -> Linking -> Other Linker Flags
#
-all_load
#
-force_load ./QHALib.a
~~~

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;让编译链增加 .a库 所有的符号表，再编译后，**nm Framework** 会看到现在有符号表都全出现。

* [IOS 温习之路 ”Other Linker Flags“](https://zhuanlan.zhihu.com/p/34232905)
* [iOS静态库中类的分类问题和符号冲突问题(Xcode other Link Flags)](https://www.jianshu.com/p/f7b0aa817cff)

### 其他 

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;当通过 Other Linker Flags 设置 Framework 之后，可能在编译时会出现其他没有符号表（Undefined symbol）的类或者函数，此时这些类大多是系统类，即需要在 Framework  -> Build Phases -> Link Binary With Libraries 上添加缺少的系统库（可以通过名字判断是哪个系统库）。这些依赖的系统库，如果第三方支持 pod，可以在 podspec 查看。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Other Linker Flags 还有如 **-lc++** 的等配置，这具体需要查看第三库给的 Demo & 说明文档进行配置。


## 访问动态库中的资源文件

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在 Framework 中使用 image/storyboard/xib 等资源，可以直接访问 Framework 中的资源！但是需要修改方式。这是由于一般 App 本地资源都是在 mainBundle 下，而 Framework 其实也是一个 bundle，所以 Framework 里面的资源是在这个 bundle 里。因此只需指定 Framework 这个 bundle 来获取就可以：

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;注意：如下方式只适合于 Dynamic Framework，而 Static Framework 不支持，因为它不是 bundle（囧），静态库的只能按全路径查找，或者将资源通过将 asset & bundle 的形式提供出去。

~~~
// 图片
NSBundle *bundle = [NSBundle bundleForClass:[self class]];
UIImage *img = [UIImage imageNamed:\*imageName*\ inBundle:bundle compatibleWithTraitCollection:nil];

// xib
NSBundle *woHomeBundle = [NSBundle bundleForClass:[ViewController class]];
ViewController *mainVC = [[ViewController alloc] initWithNibName:@"ViewController" bundle:woHomeBundle];

NSBundle *bundle = [NSBundle bundleForClass:[View class]];
View *this = [bundle loadNibNamed:@"View" owner:nil options:nil].firstObject;
~~~

* [iOS Framework 开发资源文件(图片、本地HTML、xib等)加载](https://www.jianshu.com/p/958ad743d19e)


## 多架构合并

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;通过 **lipo** 将编译出来的真机和虚拟机架构的 Framework 合并，使用 Aggregate，在

~~~
Build Phase -> + New Run Script Phase
~~~

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;其添加 Shell 脚本来运行，内容如下

~~~shell
# 输出保存合并的 Framework 文件夹路径，为当前工程文件下的 XXX-universal 文件夹
UNIVERSAL_OUTPUT=${SRCROOT}/${CONFIGURATION}-universal

# xcodebuild 创建 真机 & 虚拟 架构的 Framework
xcodebuild -configuration "${CONFIGURATION}" -target "${PROJECT_NAME}" -sdk iphoneos ONLY_ACTIVE_ARCH=NO   BUILD_DIR="${BUILD_DIR}" BUILD_ROOT="${BUILD_ROOT}" clean build
xcodebuild -configuration "${CONFIGURATION}" -target "${PROJECT_NAME}" -sdk iphonesimulator ONLY_ACTIVE_ARCH=NO BUILD_DIR="${BUILD_DIR}" BUILD_ROOT="${BUILD_ROOT}" clean build

# 删除可能输入文件路径已有的文件 & 文件夹
if [ -d "${UNIVERSAL_OUTPUT}" ]
then
rm -rf "${UNIVERSAL_OUTPUT}"
fi
mkdir -p "${UNIVERSAL_OUTPUT}"

# 将其中一个架构生成的 Framework 拷贝到 输入路径文件夹下（注意如果没有这个 Framework 是无法进行合并的）
cp -R "${BUILD_DIR}/${CONFIGURATION}-iphoneos/${PROJECT_NAME}.framework" "${UNIVERSAL_OUTPUT}/"

# 通过 lipo 进行合并
lipo -create "${BUILD_DIR}/${CONFIGURATION}-iphonesimulator/${PROJECT_NAME}.framework/${PROJECT_NAME}" "${BUILD_DIR}/${CONFIGURATION}-iphoneos/${PROJECT_NAME}.framework/${PROJECT_NAME}" -output "${UNIVERSAL_OUTPUT}/${PROJECT_NAME}.framework/${PROJECT_NAME}"

# 打开合并的 Framework 所在的文件夹
open "${UNIVERSAL_OUTPUT}"
~~~

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;注意：架构是跟支持的 iOS版本 有关，如果发现合并后需要的架构缺少，请检查项目的 Target（记住是原本 项目 Framework 的 Target，而不是 Aggregate 的 Target） 

~~~
Build Setting -> Development -> iOS Development Target: iOS8.0 
及 
Build Setting -> Architectures -> Build Active Architectures Only: NO
Build Setting -> Architectures -> Valid Architectures 包含其他编译架构
~~~

* [xcode的环境变量，Build Settings参数，workspace及联编设置](https://blog.csdn.net/bobo553443/article/details/78564628)
* [xcodebuild API](https://www.jianshu.com/p/c32263138af3)
* [【mac】lipo命令详解](https://blog.csdn.net/SoaringLee_fighting/article/details/82994510)

## framework.dSYM

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;只需要设置 Release 模式下编译就会有 .framework.dSYM