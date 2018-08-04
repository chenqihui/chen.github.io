---
title: Github Pages + Hexo 的博客
date: 2018-06-15 11:41:12
tags: 
  - Github
  - Markdown
categories:
  - 其他

---

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;最近找资料时看到的博客，然后就想到自己虽然没有独立的博客，但早前的尝试是使用 

>[GitHub Pages](https://pages.github.com/) | Websites for you and your projects, hosted directly from your GitHub repository. Just edit, push, and your changes are live.  
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;由于 Github Pages 支持 jekyll，所以当时就（时间是14年的时候）决定使用 jekyll，可是最后没用上，好像是配置问题，具体的也忘了，最后就是 jekyll 的主题没使用上，所以它就只是个简单的页面（惭愧）。而这次看到 [小鱼周凌宇のCODE_HOME](http://zhoulingyu.com/) 萌妹子的博客主题不错，再次决定动手搞起来，采用 [Hexo](https://hexo.io/zh-cn/) + [NexT](http://theme-next.iissnan.com/) 主题，当然由于没有对外的 IP，所以依然部署到 Github Pages 上。

<!-- more -->

#### Hexo

安装及使用是真方便

```shell
npm install hexo-cli -g
hexo init blog
cd blog
npm install
hexo server
```

写博客、发布文章，然后对 markdown 进行编辑即可

```shell
hexo new post "article title"
hexo g   // 生成
hexo d   // 部署
```

#### 部署 Github Pages + Hexo

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;创建 [GitHub Pages](https://pages.github.com/) 可看官网链接，搜索下也有不错的教程。只是大多文章还是依然写着控制分支到 gh-pages，可是当我部署代码前，查看项目配置发现工程已经被限制 master 分支，并且不允许修改。

![](http://pacfu36li.bkt.clouddn.com/githubpagesmaster.png?attname=)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;最开始以为跟 jekyll 一样将工程推送到 Git 仓库就可以了。但明显不是，Hexo 提供的静态生成页面和推送到仓库的命令，应该算是 Hexo 转换到 jekyll 的工程或者 Github Pages 支持的静态网页目录文件。

>安装 hexo-deployer-git。
>
>$ npm install hexo-deployer-git --save
>
>修改 hexo 项目的 _config.yml 配置。
>
>```
>deploy:
  type: git
  repo: <repository url>
  branch: [branch]
  message: [message]
>
参数	描述
repo	库（Repository）地址
branch	分支名称。如果您使用的是 GitHub 或 GitCafe 的话，程序会尝试自动检测。
message	自定义提交信息 (默认为 Site updated: {{ now('YYYY-MM-DD HH:mm:ss') }})
>```

#### 最后
我的配置如下

```xml
# Extensions
## Plugins: https://hexo.io/plugins/
## Themes: https://hexo.io/themes/
theme: next

# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy:
  type: git
  repo: git@github.com:chenqihui/chenqihui.github.io.git
  branch: master
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;还有例如名字的修改，主题的配置修改，如头像等等，我是通过比较别人在 Github 上传代码的来快速实现，也可以看看官网的说明。

参考：  
《[基于Hexo搭建博客并部署到Github Pages - 简书](https://www.jianshu.com/p/2b09156ee5b1)》

注意：

```
npm install hexo-deployer-git --save
```
输入上面的命令来发布到 Github 上，然后再执行hexo d来部署。否则会出现Deployer not found:git的错误。耐心等待，出现Deployer done: git表示你部署成功了！输入网址your_username.github.io去看看吧。一般来说如果出现莫名的问题，按照以下步骤即可解决。

```
1、删除.deploy_git文件夹
2、hexo clean
3、hexo g
4、hexo d
```


#### Markdown

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;不论是写博文，简书，或者 Github 项目的 README，都已经习惯写 Markdown 格式。所以想简单介绍下它 《[README文件语法解读，即Github Flavored Markdown语法介绍](https://github.com/chenqihui/README)》。

在线编辑阅读器：  
[欢迎使用 Cmd Markdown 编辑阅读器](https://www.zybuluo.com/mdeditor#1237871)

#### 七牛图床

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;说到文章之类，难免添加图片。而 Markdown 添加图片是采取引用发的方式（就是必须图片是在某服务器上可访问的文件）。我的 Github 项目的 README 使用 图片是保存在对应项目的 images 文件夹。可是当写博文时，除了考虑单独开个 Git 仓库来保存图片，还有其他方式，就是使用 [七牛图床](https://portal.qiniu.com/create) 。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;七牛图床还提供命令上传，cool。这样就可以通过脚本来上传图片。我影响有介绍过使用 Alfred 的 workflow 来将截图快速上传，这个后期找到再补上[TODO]。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;这里有一篇《[利用七牛做图床, python上传图片 - 简书](https://www.jianshu.com/p/7a97f3231b95)》，我还没测试。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;好了，看到这里基本准备完毕，let‘s do it.