title: Rsync文件同步
category: Linux
date: 2014-11-28 21:15:17
---

>rsync是Unix下的一款应用软件，它能同步更新两处计算机的文件与目录，并适当利用差分编码以减少数据传输。rsync中一项与其他大部分类似程序或协议中所未见的重要特性是镜像对每个目标只需要一次发送。rsync可拷贝／显示目录属性，以及拷贝文件，并可选择性的压缩以及递归拷贝。 --[维基百科](http://zh.wikipedia.org/wiki/Rsync)

<!-- more -->

Rsync有两种传输模式。第一种是通过SSH，第二种是通过Rsync服务。
通过SSH传输文件
==============

这种方式用起来比较简单，不需要什么特别的配置。举个例子，现需要将A服务器上的`/home/A/folder1`目录同步到B服务器上的`/home/B/sync`目录下，那么可以在服务器B上运行如下rsync命令：
```
rsync sshuser@A:/home/A/folder1 /home/B/sync
```