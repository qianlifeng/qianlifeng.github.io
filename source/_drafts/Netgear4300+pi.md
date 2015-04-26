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

路由器刷OpenWRT
-----
首先，咱们要把路由器刷成[openwrt](http://openwrt.org/)系统，这样才能让一切
成为可能。去openwrt官方网站下载针对4300的rom：[openwrt-ar71xx-nand-wndr4300-ubi-factory.img ](http://downloads.openwrt.org/barrier_breaker/14.07/ar71xx/nand/openwrt-ar71xx-nand-wndr4300-ubi-factory.img)。

刷机方法：
* 有线连上路由器的一个LAN口，并把电脑IP设置为192.168.1.2，掩码设成255.255.255.0
* 在 Windows 控制面板 -> 程序 -> 打开或关闭Windows功能 里面勾选 *TFTP客户端*
* 关闭路由器。用小针按住路由器后面的reset，然后打开路由器电源。刚开始会是电源灯在黄色闪烁
，然后变成绿色闪烁。此时可以松开reset
* 打开命令行，输入
```
tftp -i 192.168.1.1 put openwrt-ar71xx-nand-wndr4300-ubi-factory.img
```
如果出现传输成功的字样，说明刷机开始

* 待确认路由器重启成功后(比如可以登录[http://192.168.1.1](http://192.168.1.1))，直接关闭路由器电源等待5分钟然后再打开电源，否则可能出现 5G WiFi 不可用的问题

配置OpenWRT
----
路由器刷完之后，还不是很好用，比如默认无线是关闭的，么有中文界面等等。下面就来一个一个配置

###配置无线网络
进入Network -> Wifi 会看到两个无线（2.4G和5G）都被禁用了，点击Edit配置一下即可

###设置拨号上网
进入Network -> Interfaces -> Wan -> Edit 将Protocol改为PPPoE， 然后设置用户名密码即可

###安装Luci中文包
这一步我们需要SSH到路由器上进行，使用putty ssh上去即可。首先，更新软件源:
```
opkg update
```
注意，由于众所周知的网络原因，有时候根本更新不了。此时可以通过代理来更新。在/etc/opkg.conf中加入如下代理配置（根据你的代理来设置）
```
option http_proxy http://192.168.1.100:1080
```

然后安装中文包
```
opkg install luci-i18n-chinese
```
安装完成之后，进入luci界面。然后到System -> System -> Language and Style，将语言改成中文。

至此，路由器基本的设置已经完成。如果要求不高，设置到这里就行了。但是既然刷了OpenWRT，有几个会满足只到这里呢？

OpenWRT插件
---------
###Shadowsocks代理
* 下载[Shadowsocks安装包](http://sourceforge.net/projects/openwrt-dist/files/shadowsocks-libev/)(选择ar71xx系列)
和 [Luci app shadowsocks](http://sourceforge.net/projects/openwrt-dist/files/luci-app/shadowsocks-spec/)用于界面管理

* 使用WinSCP将文件传到路由器（注意选择SCP协议，否则会报错）
* 使用如下命令安装Shadowsocks
```
opkg install shadowsocks-libev-spec-{版本号}-{架构}.ipk
opkg install luci-app-shadowsocks-spec-{版本号}.ipk
```

* 安装完成后到Luci界面的 *服务* 菜单里面就会看到Shdowsocks选项了。界面上稍微配置配置就能FQ了，爽歪歪:)

###挂载移动硬盘

* 安装USB驱动
```
opkg update
opkg install kmod-usb-storage #安装usb存储设备驱动
opkg install kmod-usb-storage-extras
opkg install usbutils #安装了这个后可以用 lsusb
opkg install ntfs-3g #挂载NTFS
opkg install block-mount  #安装之后luci的系统->挂载点下可以直接查看挂载点信息
```

使用如下命令挂载硬盘
```
lsusb #查看usb设备  
mkdir /mnt/seagate
#加载移动硬盘，且使用big_writes等参数提高性能  
ntfs-3g /dev/sda1 /mnt/seagate -o noatime,big_writes,async
```

可以把上面最后一句加入到/etc/rc.local中实现开机自动挂载

###文件共享Samba
```
opkg install luci-app-samba
```
安装完成后，重启路由器。此时服务菜单下面会多一个 *网络共享*， 简单配置一下即可,没什么好讲的了。

###BT下载Aria2

首先，安装aria2。注意，不要从自带软件源里面安装aria2，自带的是1.18.7版本，这个版本默认不支持BT和磁力链接。所以推荐下载1.18.5版本的手动安装[aria2](http://pan.baidu.com/s/1eQiwpMi)
```
opkg install aria2_1.18.5-1_ar71xx.ipk
opkg install yaaw_all.ipk
opkg install luci-app-aria2_all.ipk
```
安装完成后，重启路由器。此时服务菜单下面会多一个 *Aria2配置*， 简单配置一下即可,也没什么好讲的。
配置完成后，打开 [http://192.168.1.1/yaaw](http://192.168.1.1/yaaw) 就可以访问了

<!-- todo:
1. 硬盘休眠
2. 广告屏蔽adbyby
6. ChinaDNS
-->

动态域名解析
---------
我们这里使用DNSPod提供的动态域名解析方案。首先，你得有一个DNSPod的域名，然后你得通过如下两个请求获得一些关于这个域名的基本信息：

获取域名列表得到 domain_id：
------------------------
```
curl -X POST https://dnsapi.cn/Domain.List -d "login_email=<your login email>&login_password=xxxx&format=json"
```

获取域名的记录列表得到 record_id：
------------------------------------
```
curl -X POST https://dnsapi.cn/Record.List -d "login_email=heddom&login_password=xxx&format=json&domain_id=11078351"
```
然后再将获得的domain_id和record_id放在下面的语句中请求
```
#!/bin/bash
curl -X POST https://dnsapi.cn/Record.Ddns -d 'login_email=xxxxxxxxx&login_password=xxxxxxx&format=json&domain_id=xxxx&record_id=xxxxxx&record_line=默认&sub_domain=your_subdomain'
```
最后把上面这段加入到Crontab中，让他定时运行即可。

有了固定了域名，下面就可以做些有意思的事情了。

#暴露树莓派的SSH端口
点击网络->防火墙->端口转发，新建一个22到22的端口转发即可

###暴露离线下载到外网

在树莓派的nginx做如下配置，这样在访问subdomain.domain.com的时候就直接是下载界面了。
```
server {
        server_name subdomain.domain.com;

        location / {
                proxy_pass   http://192.168.1.1/yaaw/;
        }
}
```
