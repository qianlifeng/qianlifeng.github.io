

首先通过如下命令安装supervisor  
`sudo apt-get install supervisor`
安装完成之后，默认的配置文件应该在`/etc/supervisor/conf.d/`下面，此后你如果想创建一个新的使用supervisor运行的程序，则在此文件夹里面添加配置文件。  
这里提供一个参考例子：  

```
[program:wox]
command = uwsgi --ini your_uwsgi_path
autorestart = true
stopsignal = QUIT
stdout_logfile = log_path
redirect_stderr = true
```  

创建完之后使用`supervisorctl start wox`启动程序即可。  


