---
title: RabbitMQ Tutorials [2] Work Queue
layout: post
author: 王召辉
catalog: true
tags:
  - rabbitmq
  - 翻译
  - rabbitmq tutorials
---

![](http://www.rabbitmq.com/img/tutorials/python-two.png)

在第一个教程中，我们编写了从命名队列发送和接收消息的程序。在这个例子中，我们将创建一个工作队列``worker queue``，它将用于在多个woker之间分配耗时的任务。

工作队列的主要思想(即:任务队列)是为了避免立即执行资源密集型任务，并且必须等待它完成。相反，我们将任务安排在稍后完成。我们将任务封装为消息并将其发送到队列中。在后台运行的一个工作进程将会弹出任务并最终执行该任务。当你在多个运行的worker任务会被他们共享。

这个概念在web应用程序中特别有用，因为在web应用程序中，它可以在短HTTP请求窗口中处理复杂的任务。

## 准备
在本教程的前一部分，我们发送了一个包含“Hello World！”的消息。现在我们将发送代表复杂任务的字符串。我们没有一个真实的任务，比如图像被缩放或被渲染的图像，所以让我们通过使用thread.sleep()函数来假装我们正在忙碌。我们将会把字符串点的数量作为它的复杂性；每个点都是工作的一秒。例如，一个假任务Hello...将会花费三秒。

我们将会稍微修改下前面的Send.java，允许命令行发送任意数量的消息。

## 循环调度

使用任务队列的优点之一是能够轻松地进行并行工作。如果我们正在建造一个积压的工作，我们可以增加更多的woker，这样就可以轻松地扩大规模。

首先，让我们尝试同时运行两个woker实例。他们都可以从队列中获取消息，但具体是怎样的呢?让我们来看看。

默认情况下，RabbitMQ将按顺序将每个消息发送给下一个消费者。平均每个消费者将得到相同数量的信息。这种分发消息的方式称为循环。

## 消息确认

完成一项任务可能需要几秒钟。你可能想知道如果一个消费者开始了一项长期的任务，并且只完成了一部分就挂掉，那么会发生什么。在我们当前的代码中，一旦RabbitMQ向客户交付一条消息，就会将其从内存中删除。在这种情况下，如果您杀死一个worker，我们将丢失它正在处理的消息。我们还将丢失发送给该特定worker尚未处理的所有消息。


但我们不想失去任何任务。如果一个woker死了，我们希望把任务交给另一个worker。

为了确保消息不会丢失，RabbitMQ支持消息确认。从customer返回一个ack(nowledgement)，告诉RabbitMQ，已经接收到一个特定的消息，并处理该消息，并且RabbitMQ可以自由地删除它。

如果客户端挂掉（它的通道关闭，连接关闭，或tcp连接丢失）并没有发送一个ACK，RabbitMq将会理解这个消息没有完全处理将会重新排队。如果在同时有其他消费者，那么它将很快地将其重新交付给另一个消费者。这样你就可以确保没有信息丢失，即使woker偶尔会挂掉。

没有任何消息超时;RabbitMQ将在worker挂掉时重新传递消息。即使处理消息需要很长时间，这也可以的。

消息答复默认是打开的。在前面的例子中我们通过``autoAck=true``标志明确关闭。是时候设置这个标记为``false``，一旦我们完成了一项任务，worker就会发送一个答复。

``` java
channel.basicQos(1); // 每次接收一个没有回复的消息 accept only one unack-ed message at a time (see below)

final Consumer consumer = new DefaultConsumer(channel) {
  @Override
  public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
    String message = new String(body, "UTF-8");

    System.out.println(" [x] Received '" + message + "'");
    try {
      doWork(message);
    } finally {
      System.out.println(" [x] Done");
      channel.basicAck(envelope.getDeliveryTag(), false);
    }
  }
};
boolean autoAck = false;
channel.basicConsume(TASK_QUEUE_NAME, autoAck, consumer);
```

使用这段代码，我们可以确定，即使在处理消息时使用ctrl+C来杀死一个woker，也不会丢失任何东西。在woker挂掉后，所有未确认的消息将被重新发送。

```
错过basicAck是一个常见的错误。这是一个很容易的错误，但后果是严重的。当你客户退出时消息将再次投递,但RabbitMQ会吃越来越多的内存不会释放任何unacked消息。
```

为了调试这类错误你可以使用``rabbitmqctl``打印``messages_unacknowledged``

``rabbitmqctl list_queues name messages_ready messages_unacknowledged``

## 消息持久性

我们已经学会了如何确保即使消费者死亡，任务也不会丢失。但是如果RabbitMQ服务器停止，我们的任务仍然会丢失。

当RabbitMQ退出或崩溃时，它将忘记队列和消息，除非您告诉它不要这样做。需要有两件事来确保消息不会丢失:我们需要标记队列和消息都是持久的。首先，我们需要确保RabbitMQ永远不会丢失我们的队列。为了实现这一目的，我们需要将其声明为持久的:

``` java
boolean durable = true; // 队列是否为持久的
channel.queueDeclare(QUEUE_NAME, durable, false, false, null);
```

尽管这个命令本身是正确的，但它在我们的目前设置中是无效的。这是因为我们已经定义了一个名为``hello``而不是持久的队列。RabbitMQ不允许您重新定义一个现有的队列，其中包含不同的参数，并将返回任何试图执行该操作的程序的错误。但是有一个快速的解决方法——让我们声明一个名称不同的队列，例如task_queue:

``` java
boolean durable = true;
channel.queueDeclare("task_queue", durable, false, false, null);
```

``queueDeclare``变化需要应用到生产和消费代码中。

此时我们确定了``task_queue``队列在rabbitmq重启时不会丢消息。现在我们需要将消息标记为持久性-通过设置``MessageProperties``为``PERSISTENT_TEXT_PLAIN``

``` java
channel.basicPublish("", QUEUE_NAME, MessageProperties.PERSISTENT_TEXT_PLAIN, message.getBytes());
```

```
注意消息的持久性

将消息标记为持久性并不能完全保证消息不会丢失。尽管它告诉RabbitMQ将消息保存到磁盘上，但是当RabbitMQ接受消息并没有保存它时，仍然有一个很短的时间窗口。而且，RabbitMQ不会为每个消息做fsync(2) —— 它可能仅保存在缓存上而不是真正的写到硬盘。持久性保证并不强大，但是对于我们的简单任务队列来说，它已经足够了。如果你需要更强的保证你可以使用[发布确认](https://www.rabbitmq.com/confirms.html)
```

## 公平保证

您可能已经注意到，调度仍然不能按照我们想要的那样工作。例如，在有两个worker的情况下，当他的基数消息很重偶数消息都很轻的时候，一个woker就会很忙，而另一个woker几乎不会做任何工作。不过，RabbitMQ并不知道这一点，而且它仍然会均匀地发送消息。

当一个消息进入队列RabbitMQ负责近负责派发消息造成的。它不会考虑customer未回复消息的数量。它仅是一味的发送第n个消息到第n个consumer。

![](http://www.rabbitmq.com/img/tutorials/prefetch-count.png)

为了避免这种情况我们可以使用``basicQos``方法设置``prefetchCount = 1``。这就告诉RabbitMQ不要一次给一个woker发送多于一条消息。或者换句话说，在处理并确认之前，不要向woker发送新消息。相反，它将把它分派给下一个不太忙的woker。

``` java
int prefetchCount = 1;
channel.basicQos(prefetchCount);
```

```
注意队列大小

如果所有的woker都很忙，你的队列就可以填满了。你会想要关注这个问题，可能会增加更多的woker，或者有其他的策略。
```

使用消息答复和prefetchCount你可以设置一个工作队列。持久性选项可以让任务存活下来即使RabbitMQ重启。
