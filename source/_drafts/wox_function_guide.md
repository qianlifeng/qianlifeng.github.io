title: Wox功能手册（中文版）
category: CSharp
date: 2014-08-10 22:53:17
Tag: [Wox]
---

[Wox]()上手指南
<!--more-->
本来想一口气直接用英文憋到Github Page上去的，想想还是先用中文酝酿一下，到时候再直接翻译过去吧。  

1. **基本使用**  
2. **系统插件** 
  * **颜色插件(Color)**
  * **控制面板插件(Control Panel)**
  * **计算器插件(Caculator)**
  * **网址插件(URL handler)**
  * **Web搜索插件(Web Searches)**
  * **命令行插件(Shell)**
  * **文件夹插件(Folder)**
  * **程序插件(Programs)**
  * **系统命令插件(System Commands)**
  * **第三方插件提示插件(Third-party Indicator)**
  * **查找文件插件(Find file)**  
  
    ![http://api.drp.io/files/544a1c854f080.gif](http://api.drp.io/files/544a1c854f080.gif)
  
    从[v1.1.0](https://github.com/qianlifeng/Wox/milestones/V1.1.0)开始，Wox自带了查找文件插件。此插件用来替换原来的[Everything插件](https://github.com/qianlifeng/Wox.Plugin.Everything)，替换的原因如下：
    1. 原来的Everything插件因为需要依赖Everything运行着才能使用，所以就显得有点鸡肋。既然已经运行着Everything，那为什么还要再来使用你的这个插件呢？
    2. 因为系统架构的原因，许多用户在使用Everything插件的时候报告了各种各样的运行错误，使用不了此插件  
    
    幸运的是，通过[https://github.com/yiwenshengmei/MyEverything](https://github.com/yiwenshengmei/MyEverything)这个项目，我们可以将.net版本的Everything带到Wox当中。目前已经基本使用了Everything的功能。但是内存占用和搜索速度上还有待进一步提高。
  
3. **第三方插件**
4. **主题**
5. **热键**
6. **代理**
7. **上下文菜单**  

  ![http://api.drp.io/files/544a2c5b85f45.png](http://api.drp.io/files/544a2c5b85f45.png)
  
  从[v1.1.0](https://github.com/qianlifeng/Wox/milestones/V1.1.0)开始，Wox实现了搜索项的上下文菜单功能。如上图所示，在选中搜索项的时候，如果在右边能看到一个小菜单的图标的话就说明这个项是有菜单的。此时，你只需使用`Shift + 回车`即可进入菜单选项。如果想从菜单界面返回到搜索项界面只需要按`Esc`即可。  
  
  ![http://api.drp.io/files/544a2d1e2758a.png](http://api.drp.io/files/544a2d1e2758a.png)  
  
  菜单项当中提供了一些你可能要对此文件/命令要进行的操作，比如使用管理员权限打开此文件或者打开文件所在目录等等。目前，菜单项只能由插件制作者添加，因为对于一个搜索项，它需要有哪些菜单，插件制作者应该最清楚。
