---
title: 安装python三方包到virtualenv
date: 2014-10-20 20:53:17
tags: [python]
---

Python 有些依赖包在 windows 需要自己编译安装，配置编译环境这些就已经够烦人的了，最关键的是环境配置对了有时候在编译的过程中还是会出现各种莫名其妙的错误。此时，一个预编译好的安装文件就显得很有必要了。

[http://www.lfd.uci.edu/~gohlke/pythonlibs/](http://www.lfd.uci.edu/~gohlke/pythonlibs/)这个网站里面就包含了很多在 windows 下面安装比较复杂的 python 三方库，包括`lxml`,`jinja2`,`psycopg`等等。如果你不用 virtualenv，那么默认下载下来直接安装就行了。

如果你需要将下载的可执行文件包安装到已经存在的 virtualenv 中，那么可以使用如下命令进行安装。

`virtualenv\script\easy_install.exe <your executable file path>`
