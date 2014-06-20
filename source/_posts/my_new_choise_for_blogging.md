title: Github+Hexo+Travis打造史上最懒博客
category: 其他
date: 2014-06-20 19:33
tags:
---

我前面写过使用[Hexo发布静态博客的文章](http://scottqian.com/2013/11/06/static-blog-start/)。静态博客抛弃了原来笨重的写作方式，不需要数据库，不需要程序来托管运行，编译后的html随便放到哪里都能显示等等，这些在我看来都是好处。  而我今天要介绍的则是将这一方式更近一步，做的更加彻底一点。  

<!--more-->

我以前使用hexo编译后的文章都是托管在我买的[DigitalOcean](https://www.digitalocean.com/?refcode=ee0f439bc35c)(小尾巴走起~~)主机上的，一直也没出过什么问题，不过话说都是html的文件，能怎么出问题呢。不过这样做有个问题就是每次发布文章的时候比较麻烦，你首先要写好title.md，然后运行`hexo generate`，最后`hexo deploy`（如果你配置了deploy的话）。在Web大行其道的今天，我希望的是最好能简化到直接打开某个网页，编辑title.md，然后保存一下就自动编译成html并显示在我的博客上了，Github+Travis让这一切成为可能。  

大概的流程是这样的：
  1. 我在Github上使用编辑Markdown文件，点击保存
  2. Travis在文件更改后自动调用Hexo编译并发布到Github
