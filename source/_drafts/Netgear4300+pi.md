title: 我的网件4300+树莓派
date: 2015-034-19 21:15:17
---

在同事的推荐下，入了WNDR4300路由器。天猫335买的，好像比某东做活动的时候还贵不少。anyway，等不到那个时候啦。
配合上次买的树莓派，可以开始折腾啦。

<!-- more -->

首先，把目标定下来，我理想中的样子应该是：
* 能够远程下载。不管是在外面还是在家里，直接在web上操作一下即可
* 小型NAS。可以多个设备访问移动硬盘中的资源，包括在线播放远程下载的电影等
* 外部直接能SSH到我的树莓派上，例如ssh pi.xxxx.com
* 路由器应该直接翻墙，这样家里所有的设备都可以透明访问被墙网站

路由器刷openwrt
-----
首先，咱们要把路由器刷成[openwrt](http://openwrt.org/)系统，这样才能让一切
成为可能。去openwrt官方网站下载针对4300的rom：[openwrt-ar71xx-nand-wndr4300-ubi-factory.img ](http://downloads.openwrt.org/barrier_breaker/14.07/ar71xx/nand/openwrt-ar71xx-nand-wndr4300-ubi-factory.img)。

openwrt插件：
1. 硬盘休眠
2. 广告屏蔽adbyby
3. shadowsocks
4. aria2
5. 网络共享 Samba
6. ChinaDNS
