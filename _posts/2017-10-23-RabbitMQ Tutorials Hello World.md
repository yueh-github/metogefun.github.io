---
title: RabbitMQ Tutorials [1] Hello World
layout: post
tags:
  - rabbitmq
  - 翻译
  - rabbitmq tutorials
---

## 介绍

RabbitMQ是一个消息代理：它接受并转发消息。你可以把它想象成邮局：当你把你想要的邮件放在一个邮筒里，你可以确定邮递员先生最终会把邮件送到你的收件人。在这个类比中，RabbitMQ是一个邮筒、邮局和邮递员。

RabbitMQ和邮局的主要区别在于，它不处理纸张，而是接受、存储和转发二进制的数据消息。

RabbitMQ和消息传递通常使用一些术语。

生产只意味着发送。发送消息的程序是一个生产者:

![producer](http://www.rabbitmq.com/img/tutorials/producer.png)

队列是位于RabbitMQ内的一个邮筒的名称。尽管消息通过RabbitMQ和您的应用程序流动，但它们只能存储在队列中。队列仅由主机的内存和磁盘限制，它本质上是一个大的消息缓冲区。许多生产者可以将消息发送到一个队列，许多消费者可以尝试从一个队列接收数据。这就是我们表示队列的方式：

![queue](http://www.rabbitmq.com/img/tutorials/queue.png)

消费与接收有相似的含义。消费者是一个主要等待接收消息的程序:

![consumer](http://www.rabbitmq.com/img/tutorials/consumer.png)

请注意生产者、消费者和代理不必在同一个主机上；实际上在大多数应用程序中它们并不在一台主机上

"Hello World"

在本教程的这一部分中，我们将用Java编写两个程序；一个发送单个消息的生产者，一个接收消息并将其打印出来的消费者。我们将忽略一些Java API的细节，集中精力在这个非常简单的事情。一个"Hello World"消息传递。

在下面的图中, "P"是我们的生产者，"C"是我们的消费者。中间的盒子是队列 —— RabbitMQ为消费者代表的一个消息缓冲区。

![](http://www.rabbitmq.com/img/tutorials/python-one.png)

```
Java客户端库
rabbitmq支持多种协议。本教程使用AMQP 0-9-1，这是一个用于消息传递的通用协议。
```

### Sending

![sending](http://www.rabbitmq.com/img/tutorials/sending.png)

我们将调动我们的消息发布者(sender)``Send``和我们的消息接受者(receiver)Recv。发布者将连接到RabbitMQ，发送一条消息，然后退出。

在``Send.java``中

``` java
public class Send {
    private final static String QUEUE_NAME = "hello";

    public static void main(String[] args) throws IOException, TimeoutException {
        Channel channel = null;
        Connection connection = null;
        try {

            ConnectionFactory factory = new ConnectionFactory();
            factory.setHost("localhost");
            connection = factory.newConnection();
            channel = connection.createChannel();

            channel.queueDeclare(QUEUE_NAME, false, false, false, null);
            String message = "Hello World!";
            channel.basicPublish("", QUEUE_NAME, null, message.getBytes());
            System.out.println(" [x] Sent '" + message + "'");

        } finally {
            if (channel != null)
                channel.close();
            if (connection != null)
                connection.close();
        }

    }
}
```

连接对套接字连接进行抽象，并负责协议版本的协商和身份验证等。这里我们连接到本地机器上的一个代理——因此是本地主机。如果我们想要连接到一台不同机器上的代理，我们只需在这里指定它的名称或IP地址。

接下来，我们创建一个channel，这是用于完成任务的大部分API都驻留在其中。

要发送，我们必须声明一个队列供我们发送到;然后我们可以将消息发布到队列:

```
channel.queueDeclare(QUEUE_NAME, false, false, false, null);
String message = "Hello World!";
channel.basicPublish("", QUEUE_NAME, null, message.getBytes());
System.out.println(" [x] Sent '" + message + "'");
```

声明队列是幂等的——它只在不存在的情况下才会被创建。消息内容是一个字节数组，因此你可以编码任何你选的格式。

最后，我们关闭这个channel和链接

```
channel.close();
connection.close();
```

### receiving

RabbitMQ推送一个消息给我的消费者，所以不像发布者发布单个消息，我们会让它运行来监听消息并将它们打印出来。

我们的消费者是由rabbitmq推送消息，因此与发布单一消息的发布者不同，我们会让它继续运行以侦听消息并将其打印出来。

![receiving](http://www.rabbitmq.com/img/tutorials/receiving.png)

``` java
public class Recv {
    private final static String QUEUE_NAME = "hello";

    public static void main(String[] args) throws IOException, TimeoutException {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");
        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();

        channel.queueDeclare(QUEUE_NAME, false, false, false, null);

        System.out.println(" [*] Waiting for messages. To exit press CTRL+C");

        Consumer consumer = new DefaultConsumer(channel) {
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope,
                                       AMQP.BasicProperties properties, byte[] body)
                    throws IOException {
                String message = new String(body, "UTF-8");
                System.out.println(" [x] Received '" + message + "'");
            }
        };

        channel.basicConsume(QUEUE_NAME, true, consumer);
    }
}
```
``DefaultCustomer``是一个实现消费者接口的类，我们将使用它来缓冲服务器推送给我们的消息。

和publisher设置相同，我们打开一个``connection``和``channel``，声明我们将要使用的队列。注意，这与发送发布到的队列匹配。

注意，我们在这里也声明了队列。因为我们可能在发布者之前启动消费者，所以我们希望在尝试使用消息之前先确保队列存在。

我们将告诉服务器将消息从队列中传递给我们。因为它将异步地推送我们的消息，所以我们提供了一个回调，以一个对象的形式来缓冲消息，直到我们准备好使用它们为止。这就是``DefaultConsumer``要做的。


