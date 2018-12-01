---
title: Github+Hexo+Travis打造史上最懒博客
date: 2014-06-20 19:33:00
tags: [随笔]
---

我前面写过使用[Hexo 发布静态博客](http://scottqian.com/2013/11/06/static-blog-start/)的文章。静态博客抛弃了原本笨重的写作方式，NO 数据库，NO Web 程序，而且生成后的 html 随便放到哪个 Server 都能显示。如果使用了数据库还必须安装 Mysql 等等之类的数据库服务，以后还得数据库备份啊，还原啊什么的，想想就头疼。所以，使用静态博客对我而言看来是个不错的选择。我今天要介绍的则是将这一方式做的更加彻底一点，摆脱手动编译的麻烦，利用`Github` + `Travis` 自动编译发布文章（程序员果然就是懒~\_~）。

我以前使用 Hexo 的时候，都是手动执行`hexo generate`后，再利用[Bit Torrent Sync]() 将 public 文件夹同步到我买的[DigitalOcean](https://www.digitalocean.com/?refcode=ee0f439bc35c)(小尾巴走起~~)主机上。虽说一直也没出过什么问题，不过总感觉这样还是不够轻量级，因为你需要在服务端和本地安装`Bit Torrent`。而我希望的是最好能简化到直接打开某个网页，新建一个 xxxx.md，编辑然后保存。此时我的博客就应该跟着更新了。经过一番探索，果然通过 Github+Travis 还是可以实现的。

# 创建 Gihutb 仓库

首先，你需要在 Github 上面创建一个新的 Repository 用来存放你的博客，名字有一定的要求，比如你的 Github 用户名是`qianlifeng`，那这个新建的 Repsoitory 必须叫`qianlifeng.github.io`。此时，你在新建的仓库里面放入一个 index.html 文件，然后访问`http://qianlifeng.github.io`就可以看到这个 index.html 的网页了。

除了默认的**master**分支，我们需要建立一个**source**分支专门用来存 hexo 放源文件。**master**分支则存放 hexo 生成之后的 html 文件。

# 启用 Travis 自动集成

Travis 是一个持续集成（Continue Integrate）的服务，对于开源的 Github 项目它是免费的。你需要到他的网站使用 Github 账户登录，然后设置页面打开我们刚才新建的那个仓库的持续集成的开关即可。

在**source**分支中新建`.travis.yml`文件用于 Travis 的一些配置。参加如下：

```yaml
branches:
only:
  - source
language: node_js
node_js:
  - "0.10"
script:
  - "hexo generate"
after_success:
  - "git clone https://$GH_TOKEN@github.com/qianlifeng/qianlifeng.github.io.git git_deploy"
  - "rm -r git_deploy/*"
  - "cp -r public/* git_deploy"
  - "cd git_deploy"
  - "git config --global push.default simple"
  - "git config --global user.name 'qianlifeng'"
  - "git config --global user.email qianlf2008@163.com"
  - "git add -A"
  - "git commit -m 'update'"
  - "git push -q"
env:
  global:
    secure: R47+1xIT3fihAM4YTO5u/AOFrpZeYndHLTBIMU49DGST6MO65+Ppyj/VHHUsJhmDXa7w83krUfhaR+BMQ1ntd3bOZQTnUyzHspKpum9XSRldoTx8FJNJgfuoKqglS25Qi8kmP9WA6IKwycYyY+6Xg1L2YfHvUbRHcfQlfayQFJ0=
```

注意其中的`$GH_TOKEN`，它的值是[Github application token](https://help.github.com/articles/creating-an-access-token-for-command-line-use/)，使用它可以避免我们直接将 github 密码暴露在外面用。具体怎么生成这个 token，可以参考[此文章](https://help.github.com/articles/creating-an-access-token-for-command-line-use/)。另外你需要将上面的 REPO 换成自己刚刚新建的即可。

# 使用[prose.io](http://prose.io/)在线写作

除了在 github page 中编辑文章，我推荐使用[prose.io](http://prose.io/)这个网站。此网站可以集成读取 github 里面的 markdown 文件，然后编辑同步，效果不错。

好了，一切就绪了，可以专心写文章了。:)
