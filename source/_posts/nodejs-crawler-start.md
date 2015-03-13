title: Nodejs写一个简单爬虫
date: 2014-03-03 21:04:17
tags: [nodejs]
---

[Nodejs](http://nodejs.org/)最近两年火的不行啊，作为半个前端（好吧，最近写公司项目写的快变成半个前端了，再下去是不是要全端了）我也来试着用nodejs写个最简单的爬虫，体验一把nodejs。

<!-- more -->

首先下载Nodejs什么的就不说了，直接下一步。Nodejs的包管理工具NPM也会同时被安装。这点做得比python好，装个pip还要自己动手。话说c#也有个nuget，好像包管理必不可少啊现在。安装好之后，使用npm安装如下两个依赖库：
```
npm install request
npm install cheerio
```
**注意，安装的时候一定要先将命令行跳到你项目文件所在的目录**，然后进行安装。否则安装之后会出现request找不到的错误。其中request是用来请求数据的，cherrio是用jquery的语法来解析html的，对于熟悉jquery的童鞋来说十分方便。
上代码:
```javascript
var request = require("request");
var cheerio = require("cheerio");

request("http://news.dbanotes.net/", function (error, response, body) {
  if (!error && response.statusCode == 200) {
        var $ = cheerio.load(body);
        $('tr td.title a').each(function () {
            console.log('%s (%s)', $(this).text(), $(this).attr('href'));
        });
  }
});
```

从这个小例子来看还是蛮简单的噢，执行结果如下：
<img src="/Images/nodejs-crawler-photo/result.png" />
