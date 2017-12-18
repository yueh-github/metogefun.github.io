---
title: spring boot调度任务cron表达式使用yml配置
layout: post
author: 王召辉
catalog: true
tags: 
 - spring boot
 - yml
 - spring schedule
---

通常在spring工程中我们会将定时任务的cron表达式配置在properties中，当项目迁移到spring boot下时，spring boot提供了更为便利的yml配置文件。关于yml的使用以及优势我们不做过多的讨论。本篇文章讨论下在yml中配置cron程序的使用问题。

因为我们使用yml文件所以之前类似于``@Schedule(${app.cron})``肯定是行不通的，yml配置根本不支持这种形式。这就比较难办了，我们怎么才能再次有效的使用el表达式赋值给我们的cron字段呢？

首先为了能获得到该值，我们需要定义一个类，并使用成员变量来接收这些值。

例如我们的yml文件内容如下：

``` yml
app:
  cron: 0 0 12 * * ?
```

我们定义一个Java Config的类来接收该配置

``` java
@Configurable
@ConfigurationProperties("app")
public class CronConfig {

    private String cron;

    @Bean
    public String cron() {
        return this.cron;
    }

    public void setCron(String cron) {
        this.cron = cron;
    }
}
```
重点就是在我们的``@Bean``上。这样我们就可以在我们的定时任务cron表达式使用该值了。

``` java
public class MySchedule {

    @Schedule(cron="#{@cron}")
    public void execute() {

    }
}
```

感觉是麻烦了一些😆，也可以使用properties、yml结合的方式。