---
title: 静态博客折腾记
date: 2013-11-06 21:15:17
---

心血来潮突然想搞起静态博客来，为什么呢？

说起博客独立之路也算走的曲折。一开始在[博客园](http://www.cnblogs.com/qianlifeng)写些文章，写着写着就想自己搞一个独立的空间，因为这样定制性高一点，适合自己折腾。后来就买了个虚拟空间，99 块钱 2 年。没错，是 99，不是 999，也不要 199，只要 99。当时穷学生，果断买下。后果你懂的，三天两头的挂机。再后来开始买国外的 VPS，当前用的是 DigitalOcean 的，5\$一个月。这个价钱说起来还是可以的，SSD 的配置。买了 VPS，因为自己懒，直接装了个 wordpress。一开始还好，后来经常出现数据库连接不上的问题，另外感觉 wordpress 有的太重量级了，自己插不上手。最后，慢慢就走上静态博客这条路。
静态博客我总结了一下，有以下几点好处：

- 轻量，如果不管静态生成引擎，那基本上就是和 html 和 js 打交道。没有后台，没有数据库，没有随之带来的各种烦恼。
- 速度快，因为完全是静态的页面，所以具体无与伦比的速度。
- 安全，因为都是单纯的静态页面，神马数据库注入啊，跨站攻击啊，都可以洗洗睡了。
- 部署方便。随便哪种类型的服务端，直接把生成后的页面仍过去就可以工作了。

目前可能够提供生成静态博客的引擎还是有不少的。

- [Jekyll](http://jekyllrb.com/)

  Jekyll 因 github 而出名，Github 的 pages 功能便是使用 Jekyll 进行编译。不过是我放弃使用 Jekyll 的原因是其奇怪的模板语法（后来看看其实也还好，不过这东西也讲究缘分的不是^\_^）

- [Pelican](http://getpelican.com)

  Pelican 是用 python 实现的生成引擎。其最大的缺点是对中文支持不够和编码的问题，经常性的冒出什么不能解码的错误。当然了，这是 python 的通病了。平常自己写写 python 脚本的时候最怕遇到这种编码问题，现在既然存在，当然是远而避之，珍惜生命。

- [Hexo](http://zespia.tw/hexo/)

  Hexo 使用 Node.js 写的，我目前使用的便是这个。最让我惊艳的不是这个引擎有多快，有多简单易用，而是据说这个引擎是台湾的一个大学生写的。知道后立马给跪了，也不知道现在有几个大学生能写出在 github star 能达到上千的作品。当然我目前也不能，虽然我已经不是大学生了，囧!

# Hexo

Hexo 主打的特点是快，和简单易用。快是因为依托了 Node.js 和强大的多线程处理能力。易用说的是配置简单，开箱即用。
安装 Hexo 只需要简单一句话：

```
$ npm install hexo -g
```

使用 Hexo 创建一个 Blog：

```
$ hexo init blog && cd blog
```

生成静态 Blog:

```
$ hexo generate
```

开启本地服务器，直接在浏览器中查看效果：

```
$ hexo serve
```

#### Update:

- 2014-06-20 [我的博客新选择](/post/my_new_choise_for_blogging/)
- 2015-05-05 Hexo 升级到 3.0 后，命令做了更改。本文提到的命令可能不再适用于新版本
