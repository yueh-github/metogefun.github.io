````---
title: RabbitMQ Tutorials [3] Publish/Subscribe
layout: post
author: 王召辉
catalog: true
tags:
  - rabbitmq
  - 翻译
  - rabbitmq tutorials
---
前面教程我们创建了一个工作队列。工作队列背后的假设是每个任务都被交付到一个woker。在这部分我们做一些完全不同的事情——我们将交付一个message到多个customer。这个模式被称为``publish/subscribe``。

为了说明这个模式，我们要建立一个简单的日志记录系统。它将由两个程序组成 —— 第一个将发送日志消息，第二个将接收并打印它们。

在我们的日志系统中，接收程序的每一个运行副本都会收到消息。这样，我们就可以运行一个接收器，并将日志引导到磁盘上;同时，我们还可以运行另一个接收器，并在屏幕上看到日志。

本质上，发布的日志消息将被广播到所有接收方。

## 交换

在本教程的前几部分中，我们从队列中发送和接收消息。现在是介绍Rabbit完整的消息传递模型的时候了。

让我们快速回顾一下上一篇教程的内容:

* 生产者是一个发送消息的用户应用程序。
* 队列是一个存储消息的缓冲区。
* 使用者是一个接收消息的用户应用程序。

RabbitMQ的消息传递模型中核心思想是生产者永远不会将任何消息直接发送到队列。实际上，很多时候producer甚至不知道消息是否会被发送到任何队列。

相反，生产者只能向交换``exchange``发送消息。交换是一件非常简单的事情。一方面，它接收来自生产者的消息，另一方面它将它们推到队列中。交换必须确切知道如何处理它接收到的消息。它应该是附加到一个特定的队列吗?它应该被附加到许多队列吗?或者应该被丢弃？这些规则是由交换类型定义的。

![exchange](http://www.rabbitmq.com/img/tutorials/exchanges.png)

有几种可用的交换类型：``direct``、``topic``,``headers``,``fanout``。我们将关注最后一个 —— fanout。我们创建一个该类型的交换，称为logs。

__``channel.exchangeDeclare("logs", "fanout");``__

fanout交换非常简单。正如您可能猜到的那样，它只向它所知道的所有队列广播它接收到的所有消息。这正是我们的logger所需要的。

```
交换列表

你可以使用rabbitmqctl列出所有的交换

sudo rabbitmqctl list_exchanges

在列表中将会有一些amq.*的交换和默认的（未命名的）交换。这些都是默认创建的，但现在你不太可能需要用它们。

匿名的交换

在本教程的前几部分中我们对交换一无所知，但仍然能够向队列发送消息。这是可能的因为我们使用的是默认的交换，我们用空字符串(“”)来标识。

回想一下我们之前发布的信息:

channel.basicPublish("", "hello", null, message.getBytes());

第一个参数是交换的名称。空字符串表示默认或匿名的交换：消息向以routingKey键指定的名称的队列发送，如果存在。

```

现在我们可以发布到我们命名的交换中

__``channel.basicPublish( "logs", "", null, message.getBytes());``__

## 临时队列

你可能还记得以前我们使用的是一个有特定名称的队列(hello和queue_name)。能够命名队列对我们来说是至关重要的 —— 我们需要把wokers指向同一个队列。当您想要在生产者和消费者之间共享队列时，给队列命名是很重要的。

但是对于我们logger来说并非如此。我们想要了解所有的日志消息，而不仅仅是它们的一个子集。我们也只对当前的消息感兴趣，而不是旧的消息。要解决这个问题，我们需要两件事。

首先，当我们连接RabbitMQ时，我们需要一个``新鲜的、空的队列``。要做到这一点，我们可以创建一个带有随机名称的队列，或者，甚至更好——让服务器为我们选择一个随机的队列名称。

其次，一旦我们断开了消费者的连接，队列就会被删除。

在Java客户端，当我们调用无参数``queueDeclare()``我们创建了一个持久的、独有的、自动删除的队列。

__``String queueName = channel.queueDeclare().getQueue();``__

queueName包含了一个随机的队列名字。它看起来像amq.gen-JzTY20BRgKO-HjmUJj0wLg.

## 绑定

![Binding](http://www.rabbitmq.com/img/tutorials/bindings.png)

我们已经创建了一个fanout交换和一个队列。现在我们需要告诉交换将消息发送到我们的队列。交换和队列之间的关系称为绑定。

__``channel.queueBind(queueName, "logs", "");``__

从现在开始，logs交换将向我们的队列追加消息。

```
绑定列表

你可以列出所有的绑定

rabbitmqctl list_bindings
```

![](http://www.rabbitmq.com/img/tutorials/python-three-overall.png)

生成日志消息的生产者程序与前面的教程没有什么不同。最重要的变化是，我们现在想要将消息发布到我们的log交换中，而不是匿名的那个。__我们需要在发送时提供一个routingKey，但是在fanout交换中它的值被忽略掉了__。

```
package cn.bcolor.example.mq.rabbitmq.ch3;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;

import java.util.concurrent.TimeoutException;

public class EmitLog {

    private static final String EXCHANGE_NAME = "logs";

    public static void main(String[] argv)
            throws java.io.IOException, TimeoutException {

        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");
        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();

        channel.exchangeDeclare(EXCHANGE_NAME, "fanout");

        String message = getMessage(argv);

        channel.basicPublish(EXCHANGE_NAME, "", null, message.getBytes());
        System.out.println(" [x] Sent '" + message + "'");

        channel.close();
        connection.close();
    }

    private static String getMessage(String[] strings) {
        if (strings.length < 1)
            return "Hello World!";
        return joinStrings(strings, " ");
    }

    private static String joinStrings(String[] strings, String delimiter) {
        int length = strings.length;
        if (length == 0) return "";
        StringBuilder words = new StringBuilder(strings[0]);
        for (int i = 1; i < length; i++) {
            words.append(delimiter).append(strings[i]);
        }
        return words.toString();
    }
}

```

如你所见，在建立连接之后我们声明了一个交换。这一步是必要的，因为发布到一个不存在的交换是被禁止的。

如果没有队列绑定到交换消息会丢失，但是对于我们来说是可以的。如果没有consumer监听我们可以安全的放弃这个消息。

``` java
package cn.bcolor.example.mq.rabbitmq.ch3;

import com.rabbitmq.client.*;

import java.io.IOException;

public class ReceiveLogs {
    private static final String EXCHANGE_NAME = "logs";

    public static void main(String[] argv) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");
        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();

        channel.exchangeDeclare(EXCHANGE_NAME, "fanout");
        String queueName = channel.queueDeclare().getQueue();
        channel.queueBind(queueName, EXCHANGE_NAME, "");

        System.out.println(" [*] Waiting for messages. To exit press CTRL+C");

        Consumer consumer = new DefaultConsumer(channel) {
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope,
                                       AMQP.BasicProperties properties, byte[] body) throws IOException {
                String message = new String(body, "UTF-8");
                System.out.println(" [x] Received '" + message + "'");
            }
        };
        channel.basicConsume(queueName, true, consumer);
    }
}

```
````