---
title: 使用Supervisor管理程序
date: 2014-07-25 18:27:17
---

人生苦短，我用 Supervisor

首先通过如下命令安装 supervisor

```
sudo apt-get install supervisor
```

安装完成之后，默认的配置文件应该在`/etc/supervisor/conf.d/`下面，此后你如果想创建一个新的使用 supervisor 运行的程序，则在此文件夹里面添加配置文件。

这里提供一个参考例子：

```
[program:name]
command = uwsgi --ini your_uwsgi_path
autorestart = true
stopsignal = QUIT
stdout_logfile = log_path
redirect_stderr = true
```

创建完之后使用如下命令启动程序即可

```
supervisorctl start name
```

Supervisor 还提供了 web 界面管理进程。  
可以在`/etc/supervisor/config.conf`里面进行配置：

```
[inet_http_server]
port = 127.0.0.1:8011
username = scott
password = yourpassword
```
