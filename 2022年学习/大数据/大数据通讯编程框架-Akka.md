#### Akka

Akka框架使用Actor编程模型实现的一套并发框架，通过异步消息完成业务处理的响应式编程



#### Actor

Actor编程模型和面向对象是平行的编程模型，Actor认为一切都是Actor，Actor之间也是通过消息传递实现复杂的功能，Actor的消息是异步的，发送给Actor之后不需要等待，通过Actor模型，可以轻松实现并发、异步、分布式编程。

实现Actor比较简单，主要是实现receive方法

```scala

class MyActor extends Actor {
  val log = Logging(context.system, this)


  def receive = {
    case "test" ⇒ log.info("received test")
    case _      ⇒ log.info("received unknown message")
  }
}
```



#### Akka原理

Akka实现异步的主要原理是，Actor之间的消息传输是通过一个收件箱Mailbox完成的，发送者Actor的消息发到接受者Actor的收件箱，接受者一个一个的串行从收件箱取消息调用自己的receive方法处理。如果消息异步发给多个Actor，就是并发编程了

![img](https://static001.geekbang.org/resource/image/26/13/269b28c63c69444dd9dcb0c3124e0713.png?wh=1920*767)



1. 发送者发送消息，调用Actor的引用ActorRef发送，ActorRef将消息放到接受者Actor的收件箱Mailbox就返回结果了
2. Akka调度代码会从Mailbox中串行读取消息，调用Actor receive方法处理
3. 处理完成后，可以调用ActorRef发送给其他的Actor的收件箱





#### Akka创建Actor方法

```

val system = ActorSystem("pingpong")

val pinger = system.actorOf(Props[Pinger], "pinger")
```



#### Actor监听网络端口

```

akka {
  actor {
    deployment {
      /sampleActor {
        remote = "akka.tcp://sampleActorSystem@127.0.0.1:2553"
      }
    }
  }
}
```



main函数调用创建方法，配置远程Props，实现receive方法，就可以得到一个远程通信的JVM进程，就可以实现Master-Slave架构通信了