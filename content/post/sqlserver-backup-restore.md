---
title: SQL Server数据的导入导出
date: 2012-02-08 10:28:00
---

这两天在用新学习的 python 抓新浪微博首页的数据，这些数据都被存在的 sql server 当中。因为白天在公司和晚上在宿舍用的数据库版本不一样，所以如果在宿舍直接附加公司的数据库的话会报错。没办法，只好通过 SQL server 中提供的 BCP 导入导出的办法来同步两个地方的数据。

这里简单记录一下如何使用 BCP 工具进行数据的导入导出。首先 BCP 是 sql server 自带的工具，所以只要你安装了 sql server 后就自带了。在 cmd 中输入 cmd，你会发现他的一些参数说明选项。具体的参数选项大家可以在具体用到的时候再去查看，这里就不一一解释了（实际上我也用的不多 ^\_^）

<img src="/Images/sqlserver-backup-restore/1.png"/>

一个最简单的导出示例。如果你有个数据库 A，里面有一个表 B，那么导出 B 数据的命令如下：

```
BCP A..B out c:\currency1.txt -c -u"sa" -p"123456"
```

注意 A 和 B 中间有两个..，如果是使用继承验证的话可以去掉-u -p 选项改为-t 就行了。下图是我执行的截图：
<img src="/Images/sqlserver-backup-restore/2.png"/>

此时如果要导入已经导出的数据，执行如下命令：

```
BCP A..B in c:\currency1.txt -c -U"sa" -P"123456"
```

可以看到与导出唯一不同的地方就是 out 换成了 in。至此，最简明 sql server 数据导入导出记录完毕！
