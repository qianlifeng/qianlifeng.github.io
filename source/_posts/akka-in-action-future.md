title: Akka in action - Future
date: 2015-09-08 16:35:17
tags: [AKKA]
---

在AKKA中，Future是一种用来接收并发操作结果的数据结构，通过Futre我们可以同步或者异步的接受Actor的返回结果。
<!--more-->

注：本文中的例子基于
#同步接收Actor返回消息  

要同步的从Actor那边接收消息，可以使用`Patterns.ask`方法
```Java
ActorSelection selection = system.actorSelection("akka.tcp://ActorSystemA@192.168.8.136:2552/user/ActorA");

Timeout timeout = new Timeout(Duration.create(5, "seconds"));
Future<Object> future = Patterns.ask(selection, "from b", timeout);
String result = (String) Await.result(future, timeout.duration());
```
