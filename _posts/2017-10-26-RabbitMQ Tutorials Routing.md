---
title: RabbitMQ Tutorials [4] Routing
layout: post
tags:
  - rabbitmq
  - 翻译
  - rabbitmq tutorials
---

在上一篇教程中我们创建了一个简单的日志系统。我们能够向许多接收者广播日志消息。

在本教程中，我们将给它添加一个特性 —— 我们只需要订阅消息的一个子集就可以了。例如，我们将能够仅将关键错误消息发送到日志文件(以节省磁盘空间)，同时仍然能够在控制台上打印所有的日志消息。

## 绑定

在前面的例子中，我们已经创建了绑定。

__``channel.queueBind(queueName, EXCHANGE_NAME, "");``__

绑定是交换和队列之间的关系。这可以简单地解读为：队列对该交换上的消息感兴趣。

绑定可以使用额外的``routingKey``参数。为了避免__basicPublish__方法中``routingKey``的困惑我们称它为``binding key``。

__``channel.queueBind(queueName, EXCHANGE_NAME, 'blank');__``

``binding key``的含义取决于交换类型。我们之前使用的``fanout``交换，简单的忽略了它的值。

## Direct交换

从上一篇教程中我们的日志系统向所有customer广播了所有消息。我们希望扩展这一功能，以允许基于其严重性的过滤消息。例如，我们可能需要一个将日志消息写入磁盘的程序，只接收严重错误，而不是在警告或信息日志消息上浪费磁盘空间。

我们使用``fanout ``交换，没有给我们很大的灵活性 —— 它只能进行盲目的广播。

我们将使用``direct``交换来代替。``direct``交换背后的路由算法非常简单 —— 一条消息被发送到binding key完全匹配消息的routing key的消息队列。

为了说明这一点，请考虑以下设置：

![](http://www.rabbitmq.com/img/tutorials/direct-exchange.png)

在该设置中，我们看到``direct``交换x有两个队列绑定。第一个队列使用绑定键orange，第二个有两个绑定，一个是black一个是green。

在这个设置中使用routing key origin的消息将会被发送的Q1队列。black、green的routing key将会被发送到Q2。其他消息将被丢弃。

## 多重绑定

![](http://www.rabbitmq.com/img/tutorials/direct-exchange-multiple.png)

用同一个绑定键绑定多个队列是完全合法的。这样的话就类似于``fanout``交换。

## 发送日志

我们将在日志系统中使用此模型。我们将会发送消息到``direct``交换。我们提供log的严重性作为一个routing key。

``` java

package cn.bcolor.example.mq.rabbitmq.ch4;

import com.rabbitmq.client.BuiltinExchangeType;
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;

public class EmitLogDirect {

    private static final String EXCHANGE_NAME = "direct_logs";

    public static void main(String[] argv) throws Exception {

        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");
        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();

        channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.DIRECT);

        String severity = getSeverity(argv);
        String message = getMessage(argv);

        channel.basicPublish(EXCHANGE_NAME, severity, null, message.getBytes("UTF-8"));
        System.out.println(" [x] Sent '" + severity + "':'" + message + "'");

        channel.close();
        connection.close();
    }

    private static String getSeverity(String[] strings) {
        if (strings.length < 1)
            return "info";
        return strings[0];
    }

    private static String getMessage(String[] strings) {
        if (strings.length < 2)
            return "Hello World!";
        return joinStrings(strings, " ", 1);
    }

    private static String joinStrings(String[] strings, String delimiter, int startIndex) {
        int length = strings.length;
        if (length == 0) return "";
        if (length < startIndex) return "";
        StringBuilder words = new StringBuilder(strings[startIndex]);
        for (int i = startIndex + 1; i < length; i++) {
            words.append(delimiter).append(strings[i]);
        }
        return words.toString();
    }
}

package cn.bcolor.example.mq.rabbitmq.ch4;

import com.rabbitmq.client.*;

import java.io.IOException;

public class ReceiveLogsDirect {

    private static final String EXCHANGE_NAME = "direct_logs";

    public static void main(String[] argv) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");
        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();

        channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.DIRECT);
        String queueName = channel.queueDeclare().getQueue();

        if (argv.length < 1) {
            System.err.println("Usage: ReceiveLogsDirect [info] [warning] [error]");
            System.exit(1);
        }

        for (String severity : argv) {
            channel.queueBind(queueName, EXCHANGE_NAME, severity);
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
