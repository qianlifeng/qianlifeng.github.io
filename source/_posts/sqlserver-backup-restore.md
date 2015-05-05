title: SQL Server数据的导入导出
date: 2012-02-08 10:28
tags: [数据库]
---

这两天在用新学习的python抓新浪微博首页的数据，这些数据都被存在的sql server当中。因为白天在公司和晚上在宿舍用的数据库版本不一样，所以如果在宿舍直接附加公司的数据库的话会报错。没办法，只好通过SQL server中提供的BCP导入导出的办法来同步两个地方的数据。

<!--more-->

这里简单记录一下如何使用BCP工具进行数据的导入导出。首先BCP是sql server自带的工具，所以只要你安装了sql server后就自带了。在cmd中输入cmd，你会发现他的一些参数说明选项。具体的参数选项大家可以在具体用到的时候再去查看，这里就不一一解释了（实际上我也用的不多 ^_^）

<img src="/Images/sqlserver-backup-restore/1.png"/>  

一个最简单的导出示例。如果你有个数据库A，里面有一个表B，那么导出B数据的命令如下：
```   
BCP A..B out c:\currency1.txt -c -u"sa" -p"123456"   
```

注意A和B中间有两个..，如果是使用继承验证的话可以去掉-u -p选项改为-t就行了。下图是我执行的截图：
<img src="/Images/sqlserver-backup-restore/2.png"/>  

 
此时如果要导入已经导出的数据，执行如下命令：
``` 
BCP A..B in c:\currency1.txt -c -U"sa" -P"123456"
```
 
可以看到与导出唯一不同的地方就是out换成了in。至此，最简明sql server数据导入导出记录完毕！
