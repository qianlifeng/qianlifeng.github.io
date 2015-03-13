title: 打包python使其成为单文件可执行程序
date: 2013-12-18 18:15:17
tags: [python]
---

本文介绍了如何使用pyinstaller打包python程序，使其成为window下的单个可执行exe文件。

<!-- more -->

##准备工作

-------------

1. 安装[PyInstaller](http://www.pyinstaller.org/) `pip install PyInstaller`，具体打包方法后面会提到。

2. 下载[UPX](http://upx.sourceforge.net/)，此程序用来压缩程序，效果蛮大。我试验的结果可以缩小一倍的程序体积。  
下载完成解压后，将其加入到环境变量中。这样PyInstaller在执行的时候可以自动调用这个程序进行压缩。

3.  下载安装[Enigma Virtual Box](http://enigmaprotector.com/en/downloads.html)。正常情况下，使用PyInstaller打包的文件会有许多依赖文件，
此程序的作用是将这些依赖和主程序进行合并，最后生成一个exe程序。


##打包

-------------

假设现在要打包p4p.py文件。直接在命令行下执行：  
```
PyInstaller p4p.py
```
相比于以前的PyInstaller，2.0以后的版本好用了不少。执行上面的命令之后，应该会出现如图一连串的信息:
<img src="/Images/pyinstaller-photo/install.png"/>

注意，如果中间出现
``` 
INFO: Executing - upx --lzma -q
 ```
类似的信息，那么就说明上面提到的UPX压缩程序起作用了。生成以后会在同目录下多一个dist的文件夹出来，里面就有我们需要的所有文件了，包括一个exe的可执行文件。

但是到了这儿还没完。因为我们的目标是单个执行程序，所以我们祭出前面介绍的`Enigma Virtual Box`。
<img src="/Images/pyinstaller-photo/EnigmaVirtualBox.png"/>
简单的实在是没什么可以说的，大家看图吧。
有一点需要注意的是右下角有个`文件选项`，点开之后可以把`压缩文件`和`退出删除`都选上，这样体积又能进一步压缩了。我最后打包的单个文件大概6M多一点，说大不大说小不小，反正还可以承受。

<span style="color:red;font-weight:bold;">2013-12-19 Update:</span>  
刚刚发现其实PyInstaller已经自带了类似于`Enigma Virtual Box`的功能。只需要在执行的时候加一个参数即可：
```
PyInstaller p4p.Py --onefile
```
