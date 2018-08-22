---
title: CocoaPods 的实践笔记
date: 2018-07-08 19:04:41
tags: 
  - CocoaPods
  - Git
categories:
  - 工具
  
---
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[CocoaPods.org](https://cocoapods.org/) 的出现极大的方便了项目的集成，第三方库的发展，特别是开源，我们不单单可以使用 framework 和 a库。不过还是很怀念 Xcode 支持插件的时候，可视化的使用 CocoaPods 的时候，不仅仅是 iOS 第三方库，还会安装 Xcode 的插件等等，不得不给个赞，而现在虽然不能再使用类似插件（对于官方版本的Xcode），但是 CocoaPods 依然是一个很方便的利器。而对于通过 CocoaPods 的维护和管理的方式，也是加快了开发组件化，开源模块的进程。所以当开源自己的模块时，支持 CocoaPods 也相对重要了，所以需要熟悉它，和基本的操作使用。接下来就先从 CocoaPods 的基本操作开始。

<!-- more -->

#### CocoaPods 基本

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;安装环境官网和网上资料都很多，在国内没有梯子的需要先执行<命令 2>，不需要翻墙可直接执行<命令 4>。

##### 安装

```shell
#1、执行以下命令查看当前源，确保已经替换成功，否则再执行以上命令。
gem sources -l

#2、把下载资源替换成国内源
gem sources --remove https://rubygems.org/
gem sources -a https://gems.ruby-china.org/

#3、再次执行 1 查看是否替换成功

#4、安装。
sudo gem install cocoapods

#5、初始化，时间稍长，文件大约600M
pod setup
```

##### 使用

```shell
#查看版本
pod --version

#添加第三方类库时，如果无论是执行 pod install 还是 pod update 都卡在了 Analyzing dependencies 不动，原因在于当执行以上两个命令的时候会升级 CocoaPods 的 spec 仓库。
pod install // 不推荐该命令

#而加一个参数可以省略这一步，然后速度就会提升不少。加参数的命令如下:
pod install --verbose --no-repo-update
pod update --verbose --no-repo-update
```

##### 更新

```shell
#更新 gem
sudo gem update --system

#安装指定版本
sudo gem install cocoapods -v 0.39.0

#更新本地 podspec 列表
pod repo update

```

##### 卸载

```shell
#1、查看 Cocoapods组件 安装的地址 (比如：/usr/local/bin/pod)
which pod

#2、移除这个组件，如（/usr/local/bin）
sudo rm -rf +地址

#3、查看 gems 的程序包
gem list

#4、删除
sudo gem uninstall cocoapods -v 版本
sudo gem uninstall cocoapods-core -v 版本
```

##### pod install 与 update 的区别：

>《cocoapods进阶》的结论  
>结论一    
>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;根据 installer.repo_update 来看，pod update 默认是更新 repo 的，而 pod install 是不更新的。
所以我们有时候 pod update 时会特别慢，就是因为在跟新 repo，特别是 CocoaPods 的官方 repo。  
>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;这时我们就常常会使用 pod update --no-repo-update 来禁止更新 repo 。而 pod install 是不需要 --no-repo-update，因为它本来就不会更新 repo。
>
>结论二  
>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; pod update podName 的时候会去 Podfile.lock 文件检查这个 pod 是否安装过，如果没有安装过会抛出异常。  
>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;但是如果直接 pod update 的话就算 Podfile.lock 中没有某个 pod，这是不会抛出异常，它会默认帮你先安装好，然后写入到 Podfile.lock 文件中。
>
>结论三  
>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;根据 installer.update 来看，pod update 默认是更新 pod 的，而 pod install 是不更新的。
但是这是相对于 pod 'SDWebImage', '~>3.8.0' 这样的写法来用的。比如原来已经安装过 3.8.0 版本，Podfile.lock 中就为 3.8.0 版本，满足 ~>3.8.0 这个条件，那么pod install的时候是是不会更新到最新版的。  
>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;但是 pod update 会更新到最新版，同时改写 Podfile.lock 中的版本号为最新版。  
>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;而 pod 'SDWebImage', '3.8.1' 这种写法的话，pod install 和 pod update 是一样的。比如 Podfile.lock 中原来为 3.8.0 版本，那么不管怎样都是不等于 '3.8.1' 的。  
>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;pod install 的时候就会重新安装 '3.8.1' 版本，同时改写 Podfile.lock 中的版本号为 '3.8.1'。

#### CocoaPods 项目

##### 创建

```shell
pod lib create “Pod项目名称”
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;接着会有设置选项来生成 Xcode 项目，每个设置之后可能出现不同的设置，不过大致如下。其中第 3 个不管选择 Yes 还是 No，都会创建 Demo。[CocoaPods Guides - Using Pod Lib Create](https://guides.cocoapods.org/making/using-pod-lib-create.html)

```shell
1、What platform do you want to use?? [ iOS / macOS ]
2、What language do you want to use?? [ Swift / ObjC ]
3、Would you like to include a demo application with your library? [ Yes / No ]
4、Which testing frameworks will you use? [ Specta / Kiwi / None ]
5、Would you like to do view based testing? [ Yes / No ]
6、What is your class prefix?
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;这样项目就会有 Example 文件夹、podName 文件夹、podName.podspec。需要重点了解下 .podspec。

##### .podspec

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;默认生成的内容如下，它包含了创建的 pod 的名称、版本、介绍等。

| 名称 | 说明 |  
| :-: | --------- |  
| name | Pod 库名称，Podfile 引用库时须使用这个名称 |  
| version | 当前 Pod 库版本，在上传 Pod 项目到 Git 时需要在 Commit 上打一个Tag，这个 Tag 就是 Pod 的 Version，Pod被引入时会根据这个 Tag 来找相对应的代码 |  
| source | Pod 项目的git地址，其他项目在引入Pod库时会去这个地址找指定的代码来下载 |  
| source_files | Pod 库对外开放的代码，只有这里指定的位置的代码才会对外开放 |  
| subspec | 当前的 spec 下再包含其他的 spec，即一个 Pod 库里可以包含多个 Pod 库，也可以引用其他第三方库及对应的资源 |  
| dependency | 依赖第三方 pod，即安装当前 pod 会同时安装依赖的 pod |  

<!--|  |  |  |  |  -->

```shell
Pod::Spec.new do |s|
  s.name             = 'podName'
  s.version          = '0.1.0'
  s.summary          = 'A short description of podName.'

# This description is used to generate tags and improve search results.
#   * Think: What does it do? Why did you write it? What is the focus?
#   * Try to keep it short, snappy and to the point.
#   * Write the description between the DESC delimiters below.
#   * Finally, don't worry about the indent, CocoaPods strips it!

  s.description      = <<-DESC
TODO: Add long description of the pod here.
                       DESC

  s.homepage         = 'https://github.com/chenqihui/podName'
  # s.screenshots     = 'www.example.com/screenshots_1', 'www.example.com/screenshots_2'
  s.license          = { :type => 'MIT', :file => 'LICENSE' }
  s.author           = { 'chenqihui' => '@.com' }
  s.source           = { :git => 'https://github.com/chenqihui/testpod.git', :tag => s.version.to_s }
  # s.social_media_url = 'https://twitter.com/<TWITTER_USERNAME>'

  s.ios.deployment_target = '8.0'

  s.source_files = 'testpod/Classes/**/*'
  
  # s.resource_bundles = {
  #   'testpod' => ['testpod/Assets/*.png']
  # }

  # s.public_header_files = 'Pod/Classes/**/*.h'
  # s.frameworks = 'UIKit', 'MapKit'
  # s.dependency 'AFNetworking', '~> 2.3'
end
```

其中 subspec 如下：

```shell
s.subspec 'SDK' do |sdk|
   sdk.source_files = '~/*.{h}'
   sdk.resource = '~/*.bundle'
   sdk.vendored_libraries = "~/*.a"
end
```

##### 使用

```shell
pod install | update
```

#### 公共 CocoaPods

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; CocoaPods 在执行 pod update 或pod install 安装 pod 库的时候，会先去 Spec repo里寻找这个库的索引文件，然后根据索引文件里面的地址来寻找Pod库的存放位置，将库中的文件下载下来，完成安装。这个 Spec repo 也是个 git 仓库，本地仓库放在 ~/.cocoapod/repo/ 文件夹，这个文件夹下的 master 文件夹即是 CocoaPods 官方的 Spec repo 在本地的分支。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;通过上述，基本创建了一个本地 CocoaPods。然后就是提供给其他人使用了。其实 CocoaPods 的原理在于建立索引，创建的 podspec 就是每个 CocoaPods 的索引目录。当我们使用官方的 cocoapods 时候，就是将这个 podspec 注册并上传到官方的 Git 仓库，也就是我们 podfile 里面最上面的 source，指向的保存众多 podspec 的仓库。

```shell
source 'https://github.com/CocoaPods/Specs.git'
```

##### 创建

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;前面创建的 pod 项目中修改确认 podspec 和项目的内容。也可以使用自己的项目工程，将 podspec 复制到自己项目的目录下，并修改好对应的文件路径等。

```shell
#1、cocoapods 注册、然后通过收到一封邮件，点击确认一下（第一次）
pod trunk register YOUR_EMAIL 'YOUR_NAME'

#2、验证 podspec 文件的合法性
pod spec lint

#3、上传 podspec
pod trunk push PodName.podspec
```

##### 使用

```shell
#1、更新本地 podspec 列表
pod repo update

#2、搜索
pod search YOUR_CocoaPod_Name

#3、仍然搜索不到，并且已经是 update 成功的话
rm ~/Library/Caches/CocoaPods/search_index.json

#4、执行上面 3 命令，清空搜索再 search
```

##### 更新

1、更新项目的代码。

2、修改 .podspec 对应的版本，并 push 到 Git 上。

3、项目打 tag [Git - 打标签](https://git-scm.com/book/zh/v1/Git-%E5%9F%BA%E7%A1%80-%E6%89%93%E6%A0%87%E7%AD%BE)，GitHub 已经支持页面打 tag。

```shell
#新建标签
git tag -a v1.4 -m 'my version 1.4'

#删除标签
git tag -d v1.4

#分享标签
git push origin v1.4
```

#### 私有 CocoaPods

##### 创建

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;如果我们要使用自己私有的Pod项目的话，同样需要建立一个私有的 Spec repo，以便 CocoaPods 知道去哪查找我们 Pod 项目的索引文件。[官方教程](https://guides.cocoapods.org/making/private-cocoapods)。  

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;首先要在 gitlab 或者其他 Git 服务器上建立一个仓库来存放 Spec repo 。然后将这个 repo 添加到本地：
	
```shell
pod repo add REPONAME http://git.xxxx.com/specrepo.git 
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;这样在~/.cocoapod/repo/即可看到一个 REPONAME 文件夹，是 Spec repo 在本地的仓库，现在需要将上面建立的Pod项目的索引文件放到这个 repo 里，Pod 项目中的 TestPrivatePods.podspec 即为项目的索引文件，通过下面的命令将文件添加到repo里：
	
```shell
pod repo push REPONAME TestPrivatePods.podspec
```
如果成功，Pod项目这时就可以使用了。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"REPONAME" 等同于 master，是可以自定义的，通过如下查看，它是关联到对应的 Git 仓库

```shell
pod repo
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;这个私有 Spec repo 库的目录是：podName -> version -> podspec

##### 使用

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;给 podfile 添加如下的私有 Spec repo 的 git仓库 source 来搜索私有 pod。（作用就是模拟官方 CocoaPods 的方式，只是搜索 podspec 是使用私有的仓库）

```shell
source 'git@git.xxxx.com:ios/pods-repo.git'
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;然后也是 pod install 就可以使用，语法一样。

##### 更新

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;更新时，按正常的公共库一样来更新 podspec 的内容之后，执行，其中 REPONAME 就是通过上面查看的
	
```shell
pod repo push REPONAME TestPrivatePods.podspec
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;这里 push 的是自己的 spec 仓库里，而不是 cocoapods 的仓库。当然，可以直接给 Git repo 仓库添加文件夹 podName -> version -> podspec，这样就跳过了 cocoapods。


#### 本地 CocoaPods

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;本地 pod 就更直接了，它其实就是没有将 pod 库 push 到官方或者私有的 Git 上去，而是源码依然在本地项目，然后通过 podspec 来依靠 CocoaPods 进行项目的管理。如果留意前面创建的 pod 的项目目录就一目了然，它就是 Development Pods 里面的内容，就像它本身就是一个新建 pod 的 Demo 一样。而它除了不用 push，其他设置、配置和操作都跟公共或者私有的 CocoaPods 项目一样。

##### 使用

```shell
# 创建 podspec
pod spec create Modules

# 修改 podspec 后进行校验，大多时候本地 pod 校验时有些警告可暂时忽略，如没有 license 等，依然可以 pod 使用
pod spec lint

# podfile，path 是 podfile 与 被 pod 的本地代码的相对路径
pod 'Modules', :path => 'Modules'
# 或
pod 'Modules', :path => '../Modules'

# 引用
import Modules
```

##### 注意

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;本地 pod 在 Swift 版下是 import 模块名称，并且需要将对外的类、方法和属性声明 public 或者 open，这样它们才会被引用，可以进入模块名称文件查看。

#### 其他

1、[iOS开发中如何管理多个版本的Cocoapods、gem](https://blog.csdn.net/jnbbwyth/article/details/51910878)

参考文章里面介绍了为项目指定使用的 cocoapods 版本，即在 Gemfile 里加入

```
gem 'cocoapods', '0.35.0'
```
实际操作使用正常 pod install 会生效，可是文中是说需要使用 bundle exec 来执行 pod 命令。

#### 参考

* [如何写一个Pod，并发布到CocoaPods上 - CSDN博客](https://blog.csdn.net/becomedragonlong/article/details/45933345#0-tsina-1-22915-397232819ff9a47a7b7e80a40613cfe1)
* [创建Podspec描述文件 - 简书](https://www.jianshu.com/p/b92fc992c745)
* [cocoapods进阶 - 简书](https://www.jianshu.com/p/5cb284934be2)
* [使用私有Cocoapods仓库 中高级用法 - 简书](https://www.jianshu.com/p/d6a592d6fced)
* [iOS开发中如何管理多个版本的Cocoapods、gem](https://blog.csdn.net/jnbbwyth/article/details/51910878)

