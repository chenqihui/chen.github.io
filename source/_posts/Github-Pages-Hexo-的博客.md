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
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;由于 Github Pages 支持 jekyll，所以当时就（时间是14年的时候）决定使用 jekyll，可是最后没用上，好像是配置问题，具体的也忘了，最后就是 jekyll 的主题没使用上，所以它就只是个简单的页面（惭愧）。而这次看到其他人的博客主题不错，再次决定动手搞起来，采用 [Hexo](https://hexo.io/zh-cn/) + [NexT](http://theme-next.iissnan.com/) 主题，当然由于没有对外的 IP，所以依然部署到 Github Pages 上。

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

![](https://anakinpublicspace-1253727175.cos.ap-chengdu.myqcloud.com/blog/githubpagesmaster.png?attname=)

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

* [基于Hexo搭建博客并部署到Github Pages](https://www.jianshu.com/p/2b09156ee5b1)

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

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;不论是写博文，简书，或者 Github 项目的 README，都已经习惯写 Markdown 格式。所以想简单介绍下它 

* [README文件语法解读，即Github Flavored Markdown语法介绍](https://github.com/chenqihui/README)

在线编辑阅读器： 
 
* [欢迎使用 Cmd Markdown 编辑阅读器](https://www.zybuluo.com/mdeditor#1237871)

#### 图床（切换到腾讯COS）

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;说到文章之类，难免添加图片。而 Markdown 添加图片是采取引用发的方式（就是必须图片是在某服务器上可访问的文件）。我的 Github 项目的 README 使用 图片是保存在对应项目的 images 文件夹。可是当写博文时，除了考虑单独开个 Git 仓库来保存图片，还有其他方式，就是使用 图床。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;好了，看到这里基本准备完毕，let‘s do it.

* [七牛图床备份](https://www.jianshu.com/p/f7ea1b23e860)

#### 加密

1、安装

```
npm install --save hexo-blog-encrypt
```

2、配置开启

找到根目录下的_config.yml文件，添加如下：

```
# Security
##
encrypt:
    enable: true
```
    
3、使用，在博文开头添加

```
---
title: Hello World
date: 2018-06-11 11:29:46
keywords: 博客文章密码
password: Hello
abstract: 密码：Hello
message:  加密文章
---
```

参考

* [hexo文章加密](https://www.jianshu.com/p/44e211829447)

#### 访问数

1、配置使用

>在 /theme/next/_config.yml 找到 busuanzi_count 
>
>将 enable 设置为 true，便可以看到页脚出现访问量，上述配置表示：
>
>site_uv 表示是否显示整个网站的UV数  
>site_pv 表示是否显示整个网站的PV数  
>page_pv 表示是否显示每个页面的PV数  

2、注意

>到hexo的themes文件夹下, 进入  
>**\themes\next\layout_third-party\analytics** 
> 
>打开: busuanzi-counter.swig  
>将src=“https://dn-lbstatics.qbox.me/busuanzi/2.3/busuanzi.pure.mini.js”  
>修改为src=“https://busuanzi.ibruce.info/busuanzi/2.3/busuanzi.pure.mini.js”

参考

* [hexo页脚添加访客人数和总访问量](https://www.jianshu.com/p/c311d31265e0)
* [Hexo Next 解决 Busuanzi 统计浏览失效 - 韦人人韦](https://blog.csdn.net/ddydavie/article/details/83020549)

#### 留言板

1、创建留言板，执行之后会在根目录下创建 ```source/guestbook/index.md```

~~~
hexo new page guestbook
~~~

2、在 [来必力](https://www.livere.com/) 注册，获取评论的安装代码

3、添加 ```issue.swig``` 文件到 ```themes/next/latout/_custom/```，将来必力的安装代码复制进去

4、修改 page.swig 文件，在```<div id="posts" class="posts-expand">```的条件语句里面添加

~~~
{% elif page.type === 'issue' %}
  {{ page.content }}
  {% include '_custom/issue.swig' %}
~~~

5、在 ```1``` 生成的 ```index.md``` 添加

~~~
---
title: 留言板
date: 2019-06-27 16:35:40
type: "issue"

---
~~~

参考：

* [如何使hexo显得自己更有逼格（二）——功能补充](https://aak1247.coding.me/hexo-next-2.html)

#### 小图标

参考

* [如何在站点中添加漂亮的小图标](https://www.jianshu.com/p/8fee92823be2?nomobile=yes)

