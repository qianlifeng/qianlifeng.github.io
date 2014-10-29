title: Wox功能手册（中文版）
category: CSharp
date: 2014-08-10 22:53:17
Tag: [Wox]
---

[Wox]()上手指南
<!--more-->
本来想一口气直接用英文憋到Github Page上去的，想想还是先用中文酝酿一下，到时候再直接翻译过去吧。  

#基本使用
-----------------------------


<br/>
#系统插件
----------------------------------
  * **程序插件(Programs)**  
    
    ![http://ww2.sinaimg.cn/large/5d7c1fa4jw1elrtwot5lmj20m807maas.jpg](http://ww2.sinaimg.cn/large/5d7c1fa4jw1elrtwot5lmj20m807maas.jpg)  
    
    把这个系统插件放到第一个介绍，足可见他的重要性。Wox最核心也是最基本的一个功能就是快速启动程序。  
    
    Wox程序插件集成了拼音检索的功能。用户可以直接用拼音或拼音首字母搜索程序，这对国人来说是一项很便利的优化。而且Wox搜索列表中的每一项都有自己的权重值。每次选择一项后，该项的权重会增大。这样下次再次搜索此项的时候，该项将会优先显示在列表前面。
    
    ![http://ww2.sinaimg.cn/large/5d7c1fa4jw1elrx1jpdrij20m807m0tg.jpg](http://ww2.sinaimg.cn/large/5d7c1fa4jw1elrx1jpdrij20m807m0tg.jpg)  
    
    Wox程序搜索列表主要从两个地方获取。  
    1. 开始菜单中的程序。  
    
      Wox会自动收集开始菜单中的程序列表并索引之后供用户进行搜索。  
      
    2. 用户自定义目录中的程序。  
      
      如果用户需要索引的程序并不在开始菜单中，Wox还提供了自定义文件夹的方式让用户指定需要索引的文件夹。具体的设置在`Setting=>Plugin=>Programs`下面：  
      
      ![http://ww4.sinaimg.cn/large/5d7c1fa4jw1elrwuw8m25j20m80go76k.jpg](http://ww4.sinaimg.cn/large/5d7c1fa4jw1elrwuw8m25j20m80go76k.jpg)
  
      在指定索引目录的同时，用户还可以指定需要索引的文件类型。默认Wox会索引以`lnk;exe;bat`为结尾的程序。用户可以通过点击`Index File suffixes`按钮进行设置。  
      
    
  <br/>
  * **颜色插件(Color)**  
  
    ![http://ww1.sinaimg.cn/large/5d7c1fa4jw1elrx7z54j7j20m803gq2y.jpg](http://ww1.sinaimg.cn/large/5d7c1fa4jw1elrx7z54j7j20m803gq2y.jpg)  
    
    当用户输入符合特定条件的颜色值的时候，Wox会告诉你这个颜色是什么。
  
  <br/>
  * **控制面板插件(Control Panel)**  
  
    ![http://ww1.sinaimg.cn/large/5d7c1fa4jw1elrx9znjv9j20m803g3ym.jpg](http://ww1.sinaimg.cn/large/5d7c1fa4jw1elrx9znjv9j20m803g3ym.jpg)  
    
    Wox还支持控制面板的搜索。例如上图所示，输入防火墙的首拼音字母后，Wox会搜索出控制面板中的防火墙选项。
  
  <br/>
  * **计算器插件(Caculator)**  
  
    ![http://ww4.sinaimg.cn/large/5d7c1fa4jw1elrxc7incnj20m803gglq.jpg](http://ww4.sinaimg.cn/large/5d7c1fa4jw1elrxc7incnj20m803gglq.jpg)  
    
    Wox内置了很强大的计算插件。直接在Wox中搜索你想计算的公式，立马就能得到计算结果。
  
  <br />
  * **网址插件(URL handler)**  
  
    ![http://ww2.sinaimg.cn/large/5d7c1fa4jw1elrxeradmpj20m803ggln.jpg](http://ww2.sinaimg.cn/large/5d7c1fa4jw1elrxeradmpj20m803ggln.jpg)  
    
    当你拷贝了一个网址想快速打开此网址的时候，打开Wox，粘贴，回车即可。
  
  <br />
  * **Web搜索插件(Web Searches)**  
  
    ![http://ww4.sinaimg.cn/large/5d7c1fa4jw1elrxsgt1drj20m80aegm7.jpg](http://ww4.sinaimg.cn/large/5d7c1fa4jw1elrxsgt1drj20m80aegm7.jpg)  
    
    Wox还提供了名为Web搜索的系统插件。所谓Web搜索，就是Wox允许用户设置一个URL，然后替换其中搜索关键字部分，从而达到快速搜索的目的。最常见的莫过于谷歌，百度搜索了。Wox内置了谷歌的Web搜索。关键字是`g`。另外在添加自定的Web搜索的时候记得把需要替换的地方换成`{q}`占位符，此占位符会在正常搜索的时候替换为你的输入。  
    
    ![http://ww1.sinaimg.cn/large/5d7c1fa4jw1elrxwxd0xwj20m80goq5o.jpg](http://ww1.sinaimg.cn/large/5d7c1fa4jw1elrxwxd0xwj20m80goq5o.jpg)
    
    在Web搜索中，Wox还提供了搜索建议的功能。如上图所示，启动了搜索建议之后，在进行web搜索的过程中，Wox会根据你的搜索关键字给出其他相关的可能的搜索。建议来源可选择谷歌或者百度。
  
  <br />
  * **命令行插件(Shell)**
  
    ![http://ww1.sinaimg.cn/large/5d7c1fa4jw1elrtig5gbyj20m8090wez.jpg](http://ww1.sinaimg.cn/large/5d7c1fa4jw1elrtig5gbyj20m8090wez.jpg)  
    
    Wox提供了可替换系统运行命令的插件（默认还是Win + R触发）。此插件相比较于原生的运行程序，提供了一下一些额外的好处：  
    1. 界面更加直观，搜索更加便利。
    
      拿`iisreset`这个例子来说，如果在传统的运行界面中搜索键入reset这个命令是得不到任何提示结果的，但是在Wox中则可以得到`iisreset`这个提示，前提是你以前运行过此命令。  
      
    2. 经常使用的5个命令将会直接显示在界面上
    
  <br/>
  * **文件夹插件(Folder)**  
    
    ![http://ww4.sinaimg.cn/large/5d7c1fa4jw1elrxk3oir4j20m803g74c.jpg](http://ww4.sinaimg.cn/large/5d7c1fa4jw1elrxk3oir4j20m803g74c.jpg) 
    
    Wox提供了文件夹书签的功能。如果你有经常需要访问的文件夹，那么你可以将它加入文件夹列表中。以后直接搜索文件夹的名字即可快速打开此文件夹了。添加的设置位于：`Settings=>Plugin=>Folder`当中。
    
  <br />
  * **系统命令插件(System Commands)**  
  
    ![http://ww1.sinaimg.cn/large/5d7c1fa4jw1elrxgqz5n1j20m803gaa4.jpg](http://ww1.sinaimg.cn/large/5d7c1fa4jw1elrxgqz5n1j20m803gaa4.jpg)  
    
    Wox集成了一些系统级别的命令。例如，注销，关机，锁定等等。具体的支持列表可以从`Settings=>Plugin=>System Command`中查看。
  
  <br />
  * **第三方插件提示插件(Third-party Indicator)**  
  
    ![http://ww2.sinaimg.cn/large/5d7c1fa4jw1elrtg0ec9kj20m8068jrk.jpg](http://ww2.sinaimg.cn/large/5d7c1fa4jw1elrtg0ec9kj20m8068jrk.jpg)
    
    如上图所示，此插件用于提示其他插件的触发关键字。
    
  <br/>
  * **查找文件插件(Find file)**  
  
    ![http://api.drp.io/files/544a1c854f080.gif](http://api.drp.io/files/544a1c854f080.gif)
  
    从[v1.1.0](https://github.com/qianlifeng/Wox/milestones/V1.1.0)开始，Wox自带了文件查找插件。此插件用来替换原来的[Everything插件](https://github.com/qianlifeng/Wox.Plugin.Everything)，替换的原因如下：
    1. 原来的Everything插件因为需要依赖Everything运行着才能使用，所以就显得有点鸡肋。既然已经运行着Everything，那为什么还要再来使用你的这个插件呢？
    2. 因为系统架构的原因，许多用户在使用Everything插件的时候报告了各种各样的运行错误，使用不了此插件  
    
    幸运的是，通过[https://github.com/yiwenshengmei/MyEverything](https://github.com/yiwenshengmei/MyEverything)这个项目，我们可以将.net版本的Everything带到Wox当中。目前已经基本使用了Everything的功能。但是内存占用和搜索速度上还有待进一步提高。
  
<br/>
#第三方插件
----------------------------

  除了系统插件和内置的插件外，Wox还提供了插件平台用于插件制作者分享自己制作的插件。[http://www.getwox.com/plugin]()  
  
  目前，Wox支持的插件语言包括但不仅限于`C#`和`Python`，用户甚至可以使用`C`,`Ruby`,`Nodejs`,`Go`等等各种语言来编写Wox插件。目前对使用C#编写的插件支持度最好，Python其次。关于如何编写Wox插件，大家有兴趣可以去看[这篇文章](http://url_to_do)中的指南。
  
<br/>
#主题
-----------------------------

  ![http://api.drp.io/files/544a461139f56.png](http://api.drp.io/files/544a461139f56.png)  
  
  Wox支持丰富的主题。用户可以在设置窗口中选择自己喜欢的主题。  
  
  此外，我们还提供了一个在线主题制作工具[ThemeBuilder](http://www.getwox.com/themebuilder)方便用户进行主题制作。在网站上配置好喜欢的主题之后，点击下载，将主题文件下载到本地之后将文件重新命名为主题的名字+xaml后缀，例如：炫酷吊炸天.xaml。然后将此主题文件放在Wox目录下面的`Themes`文件夹当中并重启Wox即可。重启后，用户即可在主题列表里面看到新增的主题了。
  
<br/>
#热键
-------------------------------------
  
  ![http://api.drp.io/files/544a4878cbb40.png](http://api.drp.io/files/544a4878cbb40.png)  
  
  作为键盘流，强大的热键支持必不可少。在Wox中，热键分为两类。一类是系统已经定义好了，用户不能更改的，比如上面提到的`Ctrl + R`，另外一类就是用户可以自定义的热键，这也是我下面介绍的重点。用好这个功能再配合插件，往往能起到事半功倍的效果。  
  
  如上图所示，自定义热键基本分类两种。  
  1. 第一种是设置Wox的主热键，即通过此热键可以激活与隐藏Wox。Wox默认的主热键是`Alt + 空格`，用户如果需要更改此设置，可以将光标放到热键框内然后直接键盘键入所需的快捷键即可。
  2. 第二种是设置自定义的插件热键。举个我经常使用的翻译热键的例子，我写了一个有道翻译的[插件](https://github.com/qianlifeng/Wox.Plugin.Youdao)默认通过`yd`关键字进行翻译，如下图所示： 
  
    ![http://api.drp.io/files/544a4b2c392df.png](http://api.drp.io/files/544a4b2c392df.png)  
  
    但是我又嫌每次都去输入这么一个`yd`比较麻烦，在我急需翻译某个单词的时候会显得十分的不便捷。这时用户便可以在这里设置一个针对`yd`查询的热键。  
  
    ![http://api.drp.io/files/544a4bed8e627.png](http://api.drp.io/files/544a4bed8e627.png)  
  
    如上图所示，添加好对应的设置之后点击`Add`即可。添加完了以后，用户通过`Alt + t`热键激活的时候，Wox会自动打开并输入`yd `，用户所需要的只是立刻输入需要进行翻译的单词。**注意，在设置热键关键字的时候，往往需要多加一个空格在后面**，例如上面的yd + 空格，因为Wox默认关键字+空格才会触发插件。  
    
    另外一个非常合适自定义热键的插件是[剪贴板插件](https://github.com/qianlifeng/Wox.Plugin.ClipboardManager)，我默认使用的是`Ctrl + Shift + v`激活，是我必不可少的插件之一。
  
<br/>
#代理
------------------------------------

  ![http://api.drp.io/files/544a4243dc812.png](http://api.drp.io/files/544a4243dc812.png)  
  
  在设置窗口中，用户可以选择为Wox设置HTTP代理。这个功能对对于一些企业用户来说可能很有必要，因为他们的网络环境都是通过代理连接的。  
  
  如果用户在这边设置了代理，那么`wpm`命令和插件都会通过此代理进行连接。注：如果插件作者在代码中没有考虑到Wox提供的代理信息，那么该插件还是不支持当前设置代理的。

<br/>
#上下文菜单
-------------------------------------

  ![http://api.drp.io/files/544a2c5b85f45.png](http://api.drp.io/files/544a2c5b85f45.png)
  
  从[v1.1.0](https://github.com/qianlifeng/Wox/milestones/V1.1.0)开始，Wox实现了搜索项的上下文菜单功能。如上图所示，在选中搜索项的时候，如果在右边能看到一个小菜单图标的话就说明这个项是有菜单的。此时，你只需使用`Shift + 回车`即可进入菜单选项。如果想从菜单界面返回到搜索项界面只需要按`Esc`即可。  
  
  ![http://api.drp.io/files/544a2d1e2758a.png](http://api.drp.io/files/544a2d1e2758a.png)  
  
  菜单项提供了一些你可能要对此文件/命令进行的操作，比如使用管理员权限打开此文件或者打开文件所在目录等等。目前，菜单项只能由插件制作者添加，因为对于一个搜索项，它需要有哪些菜单，插件制作者应该最清楚。  
  
<br/>
#相关链接
-------------------------------------
官网：[http://www.getwox.com](http://www.getwox.com)  
Github：[http://www.github.com/qianlifeng/wox](http://www.github.com/qianlifeng/wox)  
异次元介绍：[http://www.iplaysoft.com/wox.html](http://www.iplaysoft.com/wox.html)  
小众软件介绍：[http://www.appinn.com/wox-launcher/](http://www.appinn.com/wox-launcher/)  
  
