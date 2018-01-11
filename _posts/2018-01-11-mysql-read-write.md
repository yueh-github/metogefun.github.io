---
title: mysql主从分离,读写分离实现
layout: post
author: 岳浩
catalog: true
data:2018-01-10 13:00:00
tags: 
 - mysql
---


mysql的复制有很多种实现方式，复制主要解决的问题是让一台服务器的数据与其他数据库保持一致，模式大概有，主从复制，主主复制，一主多从复制等
mysql的复制基本是基于两种方式来做，一种是基于语句的复制，和基于行的复制，一般我们使用基于行的复制

基于行的复制分为3步

1：在主库上把数据更改记录到二进制日志 (Binary Log) 中

2：从库将主库的日志复制到自己的中继日志 (Relay Log) 中

3：从库读取中继日志的时间，将其重放到从库的数据库中




第一步：创建并启动两台mysql，基础配置文件如下，这是最基本的配置条件
``` mysql
    [mysqld]
    user=mysql #mysql用户
    datadir=/data/mysql #mysql数据目录
    socket=/tmp/mysql.sock
    server-id=3306 #mysql服务的唯一id，配置主从，多个数据库时必须不一样
    port=3306 #mysql启动端口 多台机器端口一定要配置不同端口
    log-bin=mysql-bin #开启二进制日志 mysql的复制主要是基于日志来做
```


第二步：在主库创建一个从库读取日志的用户，主从复制时，基本就是通过这个用户来获取最新的log，执行log记录的语句来实现数据的同步
``` mysql
    CREATE USER 'yuehao'@'localhost' IDENTIFIED BY '111111';#建立专门用于Replication的账户  DENTIFIED BY 后面是密码
    GRANT REPLICATION SLAVE ON *.* TO 'yuehao'@'192.168.0.%';#特别说明一下192.168.0.%，这个配置是指明yuehao用户所在服务器，这里%是通配符，表示192.168.0.0-192.168.0.255的Server都可以以yuehao用户登陆主服务器。如果没有使用通配符，
    而访问的服务器又不在上述配制里，那么你将无法使用该账户从你的服务器replicate主服务器.
```


第三步：从库复制设置，非常重要
``` mysql
change master to
 master_host='localhost',#master的id
 master_port=3306,#master端口
 master_user='yuehao',#用户
 master_password='111111',#密码
 master_log_file='master-bin.000002',#读取文件名
 master_log_pos=4;#从什么位置读
 
 
 #注意：上面的master_log_file,master_log_pos 去主库查看，命令：show master status;
```

第四步：启动从库，查看从库配置是否准备就绪
``` mysql
start slave;#启动从库
show slave status \G;#查看从库的配置信息，能看到从库是否就绪状态
关键参数如下：
Slave_IO_Running:yes 
Slave_SQL_Running:yes
Seconds_Behind_Master:0 #标识主从复制的延迟，小于1秒
Slave_SQL_Running_State: Slave has read all relay log; waiting for more updates
```


测试一下吧，如果你的change master to落后主库，或者是配置的有问题
可以通过下面的命令来进行修改master to的配置，记住MASTER_LOG_FILE一定要去master 去查，并且要在查之前执行flush logs;才会拿到最新的日志名和读取下标
CHANGE MASTER TO MASTER_LOG_FILE='master-bin.000006',MASTER_LOG_POS=154;


关注公众号获取更多干货文章
<img src="/img/weixin/qrcode_jishujiagou.jpg" width="100" height="100" alt="AltText" />