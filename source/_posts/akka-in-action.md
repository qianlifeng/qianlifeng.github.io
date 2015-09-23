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

如果你愿意，上面这个模型你可以当本地多线程来用。

远程Actor
---
如果使用默认配置，ActorSystem会创建基于`LocalActorRefProvider`的Actor，即只在本地传递的Actor。与此对应的，我们还可以创建一个remote的Actor，这种Actor可以被远程调用，比如从另一个JVM中。  
要启用远程Actor，需要再创建`ActorSystem`的时候传入一些额外的配置。下面以一个简单的例子做说明。假如我们有两个系统A,B。B需要调用A系统上面的Actor。  

首先在A,B的maven依赖中加入额外的akka-remote引用：
```xml
<dependency>
         <groupId>com.typesafe.akka</groupId>
         <artifactId>akka-remote_2.10</artifactId>
         <version>2.3.12</version>
</dependency>
```

在A中，启用remote并创建一个Actor：
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

在B中，同样的启用remote，但是不需要创建任何Actor，直接通过`actorSelection`直接选择A中的Actor路径即可
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

如果一切顺利，那么在A的控制台中会看到类似`Received String message: from B`的消息
