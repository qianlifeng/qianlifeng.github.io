title: 用screen管理多个SSH远程会话
date: 2013-11-24 17:15:17
tags: [linux]
---

有VPS的同学应该会经常使用[putty](http://www.chiark.greenend.org.uk/~sgtatham/putty/)进行SSH远程连接吧。新开的每个SSH都是一个session，这么做有一个缺点就是如果当你退出的时候，这个session中的任务都会跟着退出。于是screen来了。  

<!--more-->

简单来说，Screen是一个可以在多个进程之间多路复用一个物理终端的窗口管理器。Screen中有会话的概念，用户可以在一个screen会话中创建多个screen窗口，在每一个screen窗口中就像操作一个真实的telnet/SSH连接窗口那样。

#安装screen 
```
sudo apt-get install screen
```

#在screen中创建一个新的session  
```
screen -S yourSessionName
```
这样你就可以新建一个session，同时terminal自动进入到新的session环境。在这个环境里面你可以像平常一样操作各种命令，运行长时间运行的脚本等等。  

#退出session（非关闭）  
可以按ctrl + A + D。注意是按住ctrl然后A，D键。此时你又回到了普通的SSH环境，而刚才那个session仍然运行着。如果想再次进入那个session，则键入：
```
screen -r yourSessionName
```
瞧，你又还原现场了，包括你各种正在运行的脚本。

#screen的其他用法请运行：
```
screen --help
```
