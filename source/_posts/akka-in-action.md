title: Akka in action
date: 2015-09-08 16:35:17
tags: [AKKA]
---

使用AKKA构建高并发的分布式系统
<!--more-->

AKKA起步  
---
本文使用Maven作为依赖管理工具。新建一个最基本的Maven console项目，并在pom.xml中加入如下依赖：

```xml
<dependencies>
    <dependency>
        <groupId>com.typesafe.akka</groupId>
        <artifactId>akka-actor_2.10</artifactId>
        <version>2.3.12</version>
    </dependency>
</dependencies>
```

新建一个`UserActor`。Actor是一个封装了状态和行为的Java对象，Actor之间通过交换消息来进行通信。

```Java
import akka.actor.UntypedActor;
import akka.event.Logging;
import akka.event.LoggingAdapter;

public class UserActor extends UntypedActor {
    LoggingAdapter log = Logging.getLogger(getContext().system(), this);

    public void onReceive(Object message) throws Exception {
        log.info("Received String message: {}", message);
        getSender().tell("got it", getSelf());
    }
}
```

在`Main`方法中新建`ActorSystem`，并向`UserActor`发送消息

```Java
import akka.actor.ActorRef;
import akka.actor.ActorSystem;
import akka.actor.Props;

public class App {
    public static void main(String[] args) {
        final ActorSystem system = ActorSystem.create("ActorSystem");
        final ActorRef myActor = system.actorOf(Props.create(UserActor.class));
        myActor.tell("hello world", ActorRef.noSender());
    }
}
```

可以把`ActorSystem`想象成一个Actor组织，所有的Actor活动都在这个组织下进行，包括Actor的创建。 一个`ActorSystem`会创建1-N个线程来真正执行的Actor任务。`ActorSystem`的创建是一个比较消耗 资源的过程，因此最好在一个逻辑程序中只创建一个`ActorSystem`。  
`Actor`模型的设计原则是尽可能的对外界屏蔽`Actor`的具体实现，因此这里我们使用`system.actorOf`得到了一个`Actor`的引用，这样就屏蔽了Actor的内部细节。

运行此程序，如果一切顺利可以看到如下输出，代表发送消息成功：

```xml
[INFO] [09/08/2015 16:44:52.482] [ActorSystem-akka.actor.default-dispatcher-3] [akka://ActorSystem/user/$a] Received String message: hello world
```
