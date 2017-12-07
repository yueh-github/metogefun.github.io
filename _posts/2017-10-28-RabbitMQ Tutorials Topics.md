---
title: RabbitMQ Tutorials [5] Topics
layout: post
author: 王召辉
catalog: true
tags:
  - rabbitmq
  - 翻译
  - rabbitmq tutorials
---

在上一篇教程中，我们改进了我们的日志系统。我们使用``direct``交换来代替只能进行广播的``fanout``交换，获得了有选择性地接收日志的可能性。

尽管使用``direct``交换提升了我们的系统，它任然有限制 —— 它不能基于多个标准进行路由。

在我们的日志系统中，我们可能不仅要根据严重程度来订阅日志，而且还要根据发出日志的源来订阅。您可能从syslog unix工具中了解了这个概念，它基于严重程度(info/warn/crit...) 和设备(auth/cron/kern...).路由日志。

这会给我们很大的灵活性 —— 我们可能想要侦听来自'cron'的，但也可能是来自'kern'的严重错误日志。

在我们的日志系统中实现这个功能需要学习更复杂的``topic``交换。

## Topic交换

发送到topic交换的消息不能有任意的routing key —— 它必须是一个由点分隔的单词列表。单词可以是任何东西，但通常它们指定了与消息相关的一些功能。一些有效的routing key示例："stock.usd.nyse", "nyse.vmw", "quick.orange.rabbit". 可以路由键中有许多单词，最高限制为255个字节。

banding key必须也是同样的形式。topic交换背后的逻辑和``direct``相似 —— 使用特定routing key的消息发送会交付到所有绑定的匹配binding key的队列。然而，binding有两个重要的特殊情况用于绑定键:

\* 恰好被一个词代替

\# 可以被0个或多个词代替

在一个例子中解释这个很容易

![](http://www.rabbitmq.com/img/tutorials/python-five.png)

在本例中，我们将发送所有描述动物的消息。消息将使用一个由三个单词(两个点)组成的路由键被发送。路由键的第一个词将描述速度，第二个颜色和第三个物种:"<speed>.<colour>.<species>".

我们创建三个绑定：Q1使用"\*.orange.*"绑定key绑定并且Q2使用"\*.\*.rabbit" and "lazy.#".

这些绑定可以概括为:

* Q1对所有橙色的动物感兴趣
* Q2想要听到关于兔子的一切和所有懒惰的动物

使用routing key "quick.orange.rabbit"的消息将会交付到两个队列。消息"lazy.orange.elephant"同样也会进入他们两个。另一方面"quick.orange.fox"将会进入第一个队列，"lazy.brown.fox"会进入第二个队列，"lazy.pink.rabbit"会进入第二个队列一次尽管他匹配两个绑定。"quick.brown.fox"不会匹配任何绑定所以他将会被丢弃。

如果我们违反合同发送一个或四个单词的消息，像 "orange" or "quick.orange.male.rabbit"会发生什么？这些消息不匹配任何绑定，将会丢弃。

另一方面"lazy.orange.male.rabbit", 尽管它有四个单词，将匹配最后一个绑定并交付到第二个队列。

```
Topic交换

topic交换功能强大可以表现的跟其他交换一样。

当一个队列绑定到# banding key —— 它将会接收所有消息，不管routing key是什么 —— 像fanout交换。

当指定特殊字符\*并且\#不用于绑定，topic交换将会和direct交换表现一样。
```

## 整合

我们将在日志系统中使用``topic``交换。我们从一个工作假设开始，假设日志的routing key会有两个词： "<facility>.<severity>".

``` java
import com.rabbitmq.client.*;

import java.io.IOException;

public class EmitLogTopic {

    private static final String EXCHANGE_NAME = "topic_logs";

    public static void main(String[] argv)
                  throws Exception {

        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");
        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();

        channel.exchangeDeclare(EXCHANGE_NAME, "topic");

        String routingKey = getRouting(argv);
        String message = getMessage(argv);

        channel.basicPublish(EXCHANGE_NAME, routingKey, null, message.getBytes());
        System.out.println(" [x] Sent '" + routingKey + "':'" + message + "'");

        connection.close();
    }
    //...
}

import com.rabbitmq.client.*;

import java.io.IOException;

public class ReceiveLogsTopic {
  private static final String EXCHANGE_NAME = "topic_logs";

  public static void main(String[] argv) throws Exception {
    ConnectionFactory factory = new ConnectionFactory();
    factory.setHost("localhost");
    Connection connection = factory.newConnection();
    Channel channel = connection.createChannel();

    channel.exchangeDeclare(EXCHANGE_NAME, "topic");
    String queueName = channel.queueDeclare().getQueue();

    if (argv.length < 1) {
      System.err.println("Usage: ReceiveLogsTopic [binding_key]...");
      System.exit(1);
    }

    for (String bindingKey : argv) {
      channel.queueBind(queueName, EXCHANGE_NAME, bindingKey);
    }

    System.out.println(" [*] Waiting for messages. To exit press CTRL+C");

    Consumer consumer = new DefaultConsumer(channel) {
      @Override
      public void handleDelivery(String consumerTag, Envelope envelope,
                                 AMQP.BasicProperties properties, byte[] body) throws IOException {
        String message = new String(body, "UTF-8");
        System.out.println(" [x] Received '" + envelope.getRoutingKey() + "':'" + message + "'");
      }
    };
    channel.basicConsume(queueName, true, consumer);
  }
}

```