title: Akka in action date: 2015-09-08 16:35:17

# tags: [AKKA]
使用AKKA构建高并发的分布式系统 <!--more-->

#AKKA起步 本文使用Maven作为依赖管理工具。新建一个最基本的Maven console项目，并在pom.xml中加入如下依赖：

```xml
<dependencies>
    <dependency>
        <groupId>com.typesafe.akka</groupId>
        <artifactId>akka-actor_2.10</artifactId>
        <version>2.3.12</version>
    </dependency>
</dependencies>
```

新建一个`UserActor`。AKKA是基于消息来进行通信的，每个Actor都可以接受到来自其他Actor的消息，然后对消息进行处理。  

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

运行此程序，如果一切顺利可以看到如下输出，代表发送消息成功：
```xml
[INFO] [09/08/2015 16:44:52.482] [ActorSystem-akka.actor.default-dispatcher-3] [akka://ActorSystem/user/$a] Received String message: hello world
```
