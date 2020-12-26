---
title: 用screen管理多个SSH远程会话
date: 2013-11-24 17:15:17
---

有 VPS 的同学应该会经常使用[putty](http://www.chiark.greenend.org.uk/~sgtatham/putty/)进行 SSH 远程连接吧。新开的每个 SSH 都是一个 session，这么做有一个缺点就是如果当你退出的时候，这个 session 中的任务都会跟着退出。于是 screen 来了。

简单来说，Screen 是一个可以在多个进程之间多路复用一个物理终端的窗口管理器。Screen 中有会话的概念，用户可以在一个 screen 会话中创建多个 screen 窗口，在每一个 screen 窗口中就像操作一个真实的 telnet/SSH 连接窗口那样。

# 安装 screen

```
sudo apt-get install screen
```

# 在 screen 中创建一个新的 session

```
screen -S yourSessionName
```

这样你就可以新建一个 session，同时 terminal 自动进入到新的 session 环境。在这个环境里面你可以像平常一样操作各种命令，运行长时间运行的脚本等等。

# 退出 session（非关闭）

可以按 ctrl + A + D。注意是按住 ctrl 然后 A，D 键。此时你又回到了普通的 SSH 环境，而刚才那个 session 仍然运行着。如果想再次进入那个 session，则键入：

```
screen -r yourSessionName
```

瞧，你又还原现场了，包括你各种正在运行的脚本。

# screen 的其他用法请运行：

```
screen --help
```
