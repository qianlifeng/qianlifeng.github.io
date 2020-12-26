---
title: Akka in action
date: 2015-09-08 16:35:17
---

# AKKA 起步

本文使用 Maven 作为依赖管理工具。新建一个最基本的 Maven console 项目，并在 pom.xml 中加入如下依赖：

```xml
<dependencies>
    <dependency>
        <groupId>com.typesafe.akka</groupId>
        <artifactId>akka-actor_2.10</artifactId>
        <version>2.3.12</version>
    </dependency>
</dependencies>
```

新建一个`UserActor`。Actor 是一个封装了状态和行为的 Java 对象，Actor 之间通过交换消息来进行通信。

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

可以把`ActorSystem`想象成一个 Actor 组织，所有的 Actor 活动都在这个组织下进行，包括 Actor 的创建。 一个`ActorSystem`会创建 1-N 个线程来真正执行的 Actor 任务。`ActorSystem`的创建是一个比较消耗 资源的过程，因此最好在一个逻辑程序中只创建一个`ActorSystem`。  
`Actor`模型的设计原则是尽可能的对外界屏蔽`Actor`的具体实现，因此这里我们使用`system.actorOf`得到了一个`Actor`的引用，这样就屏蔽了 Actor 的内部细节。

运行此程序，如果一切顺利可以看到如下输出，代表发送消息成功：

```xml
[INFO] [09/08/2015 16:44:52.482] [ActorSystem-akka.actor.default-dispatcher-3] [akka://ActorSystem/user/$a] Received String message: hello world
```

如果你愿意，上面这个模型你可以当本地多线程来用。

# 远程 Actor

如果使用默认配置，ActorSystem 会创建基于`LocalActorRefProvider`的 Actor，即只在本地传递的 Actor。与此对应的，我们还可以创建一个 remote 的 Actor，这种 Actor 可以被远程调用，比如从另一个 JVM 中。  
要启用远程 Actor，需要再创建`ActorSystem`的时候传入一些额外的配置。下面以一个简单的例子做说明。假如我们有两个系统 A,B。B 需要调用 A 系统上面的 Actor。

首先在 A,B 的 maven 依赖中加入额外的 akka-remote 引用：

```xml
<dependency>
         <groupId>com.typesafe.akka</groupId>
         <artifactId>akka-remote_2.10</artifactId>
         <version>2.3.12</version>
</dependency>
```

在 A 中，启用 remote 并创建一个 Actor：

```Java
import akka.actor.ActorSystem;
import akka.actor.Props;
import com.typesafe.config.Config;
import com.typesafe.config.ConfigFactory;

public class App {
    public static void main(String[] args) {
        Config combined = ConfigFactory.load();

        Config remote = ConfigFactory.parseString("akka.actor.provider=akka.remote.RemoteActorRefProvider");
        combined = remote.withFallback(combined);

        //配置监听地址和端口，hostname可以忽略，默认为本机地址。port如果填写0，则为随机端口
        Config addressConfig = ConfigFactory.parseString("akka.remote.netty.tcp={hostname=192.168.8.136, port=2552}");
        combined = addressConfig.withFallback(combined);

        final ActorSystem system = ActorSystem.create("ActorSystemA", combined);

        //创建名字为ActorA的Actor
        system.actorOf(Props.create(UserActor.class), "ActorA");
    }
}
```

在 B 中，同样的启用 remote，但是不需要创建任何 Actor，直接通过`actorSelection`直接选择 A 中的 Actor 路径即可

```Java
import akka.actor.ActorRef;
import akka.actor.ActorSelection;
import akka.actor.ActorSystem;
import com.typesafe.config.Config;
import com.typesafe.config.ConfigFactory;

public class App {
    public static void main(String[] args) {
        Config combined = ConfigFactory.load();

        Config remote = ConfigFactory.parseString("akka.actor.provider=akka.remote.RemoteActorRefProvider");
        combined = remote.withFallback(combined);

        //配置监听地址和端口，hostname可以忽略，默认为本机地址
        Config addressConfig = ConfigFactory.parseString("akka.remote.netty.tcp={hostname=192.168.8.136, port=2553}");
        combined = addressConfig.withFallback(combined);

        final ActorSystem system = ActorSystem.create("ActorSystemB", combined);
        ActorSelection selection = system.actorSelection("akka.tcp://ActorSystemA@192.168.8.136:2552/user/ActorA");
        selection.tell("from B", ActorRef.noSender());
    }
}
```

如果一切顺利，那么在 A 的控制台中会看到类似`Received String message: from B`的消息
