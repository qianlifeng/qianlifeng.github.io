---
title: 开启unbuntu上3306远程连接
date: 2013-11-22 21:15:17
---

1. 创建一个新的用户，并赋予权限。

```
mysql> grant all PRIVILEGES on *.* to user@'%' identified by 'pwd';
```

    把上面的user和pwd换成你自己的用户名和密码。可以通过如下命令查看是否添加成功。

```
mysql> use information_schema
mysql> select * from user_privileges;
```

    查询到有下面的结果：'user'@'%'，说明mysql已经授权远程连接。

2. 在`/etc/mysql/my.cnf`找到`bind-address`，将其配置的 127.0.0.1，直接改为你**服务器**的地址。

3. 如果还不能访问，可以查看 3306 端口是否开启。如果开启了，应该会有结果返回。

```
netstat -an |grep 3306
```
