title: "利用aria2实现在树莓派上的远程下载"
date: 2015-03-22 19:24:42
tags: [树莓派]
---

aria2是一个BT下载工具，再利用树莓派就可以实现24x7的BT下载啦

<!--more-->

首先，安装aria2。注意，如果直接通过`apt-get install aria2`安装可能会安装比较旧的版本，所以最好还是
```
sudo add-apt-repository ppa:t-tujikawa/ppa
sudo apt-get update
sudo apt-get install aria2
```
