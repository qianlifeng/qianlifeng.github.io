---
title: nginx二级域名配置小结
date: 2013-11-08 21:15:17
tags: [nginx]
---

一直就听说 nginx 很牛逼的样子，这次就在 VPS 上面体验了一把。平常一直混迹于 window 平台下，所以最多也就配配 IIS，而且仅仅限于 UI 这里点点那里点点。使用 nginx 完全是不同的感受，所有配置完全写在配置文件里面，所有东西完全掌控在你手上。极客小伙伴们快来体验一把！

为了以后查阅起来方便，我决定还是从安装开始写，说不定时间一长我连怎么安装配置 nginx 都会记不得了，正好也给新手一个完全指南。注：我使用的 VPS 系统是 Ubuntu

# nginx 安装

```
sudo apt-get install nginx
```

too young too simple 啊。装完了之后，有几个常用的目录我们需要认识一下：

1. 下面这个路径是存放 domain 配置的地方。默认下面应该有个 default 文件。

```
/etc/nginx/site-available
```

如果你有不同的域名 a.com,b.com。那么你就可以在这下面建立 a,b 文件分别对这两个域名进行配置，简单的分类作用吧。我们下面要讲到的规则基本上都是在这里面配置的。

2. 这个是 nginx 默认的日志存放路径

```
/var/log/nginx/
```

这里面基本上有两类文件：错误日志`error.log`,访问日志`access.log`。后期再配置二级域名的时候，如果配置的不对，过来查看错误日志是很有效的错误定位方法。

# 配置二级域名

打开上文提到的`/etc/nginx/site-available/default`文件。键入如下代码（scottqian.com 换成你的域名）：

```
server {
        index index.html;
        server_name ~^(?<subdomain>.+)\.scottqian\.com$;
        root /home/web/$subdomain/;

        if ($host != 'www.scottqian.com'){
                rewrite ^/(.*)$ http://www.scottqian.com/$1 permanent;
        }

        location / {
                try_files $uri $uri/ =404;
        }

}

```

注意看`server_name`那一行。这是一个正则表达式，它匹配所有**.scottqian.com 的域名，并把**代表的字符用 subdomain 变量表示。紧接着在下面使用`root`命令，将当前 url 的根地址指向`/home/web/$subdomain/`。通过这个方式实现了任意多个二级域名的配置，比如你想在又多了一个域名 lab.scottqian.com。那么只需要在`/home/web`下面建立一个 lab 文件夹即可。完全不必再来更改配置。

上面那个 if 判断语句是用来将`scottqian.com`这样的域名重定向为`www.scottqian.com`。暂时还不能去掉这个判断，因为没有这个的话 subdomain 变量就是空的，到时候根目录就是`/home/web//`，显然是不对的，我就在这上面吃过亏。
