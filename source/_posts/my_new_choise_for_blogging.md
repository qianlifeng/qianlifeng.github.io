title: Github+Hexo+Travis打造史上最懒博客
category: 其他
date: 2014-06-20 19:33
tags:
---

我前面写过使用[Hexo发布静态博客](http://scottqian.com/2013/11/06/static-blog-start/)的文章。静态博客抛弃了原本笨重的写作方式，NO数据库，NO Web程序，而且生成后的html随便放到哪个Server都能显示。如果使用了数据库还必须安装Mysql等等之类的数据库服务，以后还得数据库备份啊，还原啊什么的，想想就头疼。所以，使用静态博客对我而言看来是个不错的选择。我今天要介绍的则是将这一方式做的更加彻底一点，摆脱手动编译的麻烦，利用Github+Travis自动发布编译文章（程序员果然就是懒~_~）。

<!--more-->

我以前使用Hexo的时候，都是手动执行`hexo generate`后，再利用[Bit Torrent Sync]() 将public文件夹同步到我买的[DigitalOcean](https://www.digitalocean.com/?refcode=ee0f439bc35c)(小尾巴走起~~)主机上。虽说一直也没出过什么问题，不过总感觉这样比较麻烦。不符合我要随时随地写的要求。在Web大行其道的今天，我希望的是最好能简化到直接打开某个网页，新建一个xxxx.md，然后保存一下就自动发布在我的博客上了。当然，经过一番探索，果然通过Github+Travis还是可以实现的。 

首先，你需要在Github上面创建一个新的Repository用来存放你的博客，名字有一定的要求，比如你的Github用户名是`qianlifeng`，那这个新建的Repsoitory必须叫`qianlifeng.github.io`。此时，你在新建的仓库里面放入一个index.html文件，然后访问`http://qianlifeng.github.io`就可以看到这个index.html的网页了。  
当然，上面说的只是基础，我们需要做的是如何让hexo自动生成的public的文件上传到这个仓库下面。答案是Travis。  

Travis是一个持续集成（Continue Integrate）的服务，对于开源的Github项目它是免费的。你需要到他的网站使用Github账户登录，然后设置页面打开我们刚才新建的那个仓库的持续集成的开关即可。
xxxxxxxxxxxxxxxxx



