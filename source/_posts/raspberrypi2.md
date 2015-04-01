title: "树莓派2"
date: 2015-03-13 19:30:42
tags: [树莓派]
---

买的树莓派2终于到啦，这里介绍一下基本的上手准备工作。

<!--more-->

无图无真相，上图
<img src="/Images/raspberrypi/pi.jpg" style="width:360px"/>

#无线网络配置

拿到手第一件事情就是把无线网给配置起来，这样以后就不用网线了。首先通过有线SSH上去，然后运行命令：
```
lsusb
```
看看你的无线模块是不是已经被系统识别。如果出现最后一行信息，则说明已成功识别
```
Bus 001 Device 002: ID 0424:9514 Standard Microsystems Corp.
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
Bus 001 Device 003: ID 0424:ec00 Standard Microsystems Corp.
Bus 001 Device 004: ID 0bda:8176 Realtek Semiconductor Corp. RTL8188CUS 802.11n WLAN Adapter
```
然后编辑如下文件
```
sudo vi /etc/network/interfaces
```

在这个文件里可以配置网络相关的配置。我这里是使用的静态IP，这样方便以后SSH上去
```
auto lo

iface lo inet loopback
iface eth0 inet dhcp

auto wlan0
allow-hotplug wlan0
iface wlan0 inet static

wpa-ssid "网络名（大小写敏感）"
wpa-psk "密码"
address 192.168.1.200
netmask 255.255.255.0
gateway 192.168.1.1
```
配置成功后，重启树莓派即可
