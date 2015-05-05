title: EasyVS - Visual Studio扩展
date: 2012-01-09 21:52
tags: [DotNet]
---

前前后后研究VSX也一个多月了，这两天终于做了一个小的插件EasyVS。该插件目前只支持VS2010，vs2008的支持可能要过一段时间。还好我没有使用MEF的内容，所以移植到vs2008上应该困难不大。写这个插件的主要目的希望像Resharper那样提供许多实用的功能，让在vs中进行编码成为一种享受。

<!--more-->

#功能介绍
-------------
##Quick Region
所谓快速region功能，就是在代码视图里面通过快捷键Ctrl+Q,Ctrl+R快捷键将代码自动分类到不同的region下，目前的region包括“变量”，“构造函数”，“事件”，“方法”，“属性”等。下面是一个例子演示，这个类是我从随便翻出来的，可以看到他的代码格式很乱。  
<img src="/Images/easyvs/1.png"/>
下面通过EasyVS提供的快速Region功能进行代码的自动分类  

<img src="/Images/easyvs/2.png"/>

整理后的代码  

<img src="/Images/easyvs/3.png"/>

说白了就是让一些region的功能让程序帮你做了，省时省力。不过有一个缺点也很明显就是使用这个工具格式化出的region都一样，就缺少了自己特定的region了（原来自己的reigon会消失，以后的版本打算增加保留自己自定义region的功能）。

##Less Tab
通过设置指定的tab数量，插件能够自动为您关闭多余的Tab，减少VS内存占用，还您一个清爽的VSTab栏。例如，我在设置里面设置了只打开5个Tab

<img src="/Images/easyvs/4.jpg"/>  

那么以后在VS中你能同时打开的tab数不会超过5个。这样能够减少不知不觉中打开的Tab数，关闭不必要的tab以释放占用的内存。至于哪些tab会被关闭。你使用的越频繁的tab越不会被关闭。而很长时间没有使用的tab则关闭的几率会比较高。一句话，这个东西不会影响你正常的代码操作。

<img src="/Images/easyvs/5.jpg"/>  

<br />
#更新日志
-------------
##V0.2
  1. 增加Quick Region自定义region名字功能
  2. 修改Quick Region的分组规则，对于已经存在的region不处理。如果设置的region在当前代码中已经存在，则将外部的此类型加入已经存在的region中
  3. 增加Less Tab功能
  4. 增加自动更新功能
  5. 修复Quick Region的一个BUG（如果存在region嵌套，则出错的问题）
  6. 增加网络代理设置

##V0.1
  1.增加Quick Region功能

<br />
#下载
-------------
该插件我目前已经发布到了微软的官方VS插件库，该插件的地址为 [http://visualstudiogallery.msdn.microsoft.com/7310649d-87d9-45d2-b7da-99e5b001549e](http://visualstudiogallery.msdn.microsoft.com/7310649d-87d9-45d2-b7da-99e5b001549e)
<img src="/Images/easyvs/6.png"/>  
