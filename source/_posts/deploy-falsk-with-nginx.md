title: 在nginx上配置flask网站
date: 2014-01-14 21:46:17
tags: nginx
---

好记性不如烂笔头，前端时间配置这个玩意儿的时候本以为自己已经记得的了。最近再配置还是忘记的一干二净，所以还是记录一下以备后用。
<!--more-->

#nginx配置

----------

可以在`/etc/nginx/sites-available/`目录下新建一个config文件，这样便于不同程序之间的分隔，建立完后使用软连接链接到`/etc/nginx/sites-enable/`下面。
注意这里使用ln -s 的时候两个路径必须要使用绝对路径，不要使用相对路径。
```
server {
    server_name www.xxx.com;

    error_log /home/error.log;

    location / {
        try_files $uri @web;
    }

    location @web {
        include uwsgi_params;
        uwsgi_pass 127.0.0.1:8090;
        uwsgi_param UWSGI_CHDIR /home/pythonProject;
        uwsgi_param UWSGI_MODULE application;
        uwsgi_param UWSGI_CALLABLE app;
    }
}
```
完成之后重启nginx。
```
service nginx restart
```

运行网站的命令是：`uwsgi -s :8090 -w application:app`。

据网上的资料说还应该配置`/etc/uwsgi/apps-available`下面的内容，但是好像我没配置也正确运行了。各位看官请自己甄别。
