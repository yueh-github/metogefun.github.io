---
title:  CentOS 7下配置Redis开机启动
layout: post
author: 王召辉
catalog: true
tags:
 - redis
 - centos
---

首先在官网下载源码包，解压后进入该目录执行``make install``。redis相关命令会存在于``/usr/local/bin/``下。

启动脚本在redis home下的utils目录下：__redis\_init_script__。可以简单看下里面的内容：

```
#!/bin/sh
#
# Simple Redis init.d script conceived to work on Linux systems
# as it does use of the /proc filesystem.

REDISPORT=6379
EXEC=/usr/local/bin/redis-server
CLIEXEC=/usr/local/bin/redis-cli

PIDFILE=/var/run/redis_${REDISPORT}.pid
CONF="/etc/redis/${REDISPORT}.conf"

case "$1" in
    start)
        if [ -f $PIDFILE ]
        then
                echo "$PIDFILE exists, process is already running or crashed"
        else
                echo "Starting Redis server..."
                $EXEC $CONF
        fi
        ;;
    stop)
        if [ ! -f $PIDFILE ]
        then
                echo "$PIDFILE does not exist, process is not running"
        else
                PID=$(cat $PIDFILE)
                echo "Stopping ..."
                $CLIEXEC -p $REDISPORT shutdown
                while [ -x /proc/${PID} ]
                do
                    echo "Waiting for Redis to shutdown ..."
                    sleep 1
                done
                echo "Redis stopped"
        fi
        ;;
    *)
        echo "Please use start or stop as first argument"
        ;;
esac
```
可以看到配置文件中指定的端口号``REDISPORT``为默认的__6379__；``redis-server``命令的全路径__ EXEC__；以及配置文件的路径``CONF``。我们将该文件复制到``/etc/init.d``目录下

``` shell
cp redis_init_script /etc/init.d/redis
```

将配置文件放到``CONF``锁在的目录，``/etc/redis``是不存在的所以我们首先要创建这个文件。

``` shell
mkdir -p /etc/redis
cp  redis.conf /etc/redis/6379.conf
```

并将配置文件中的``daemonize``改为``yes``。

这个时候我们执行``service redis start``，redis服务就会启动个，会有类似于如下信息打印：

```
3822:C 30 Oct 16:45:55.129 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
3822:C 30 Oct 16:45:55.129 # Redis version=4.0.2, bits=64, commit=00000000, modified=0, pid=3822, just started
3822:C 30 Oct 16:45:55.129 # Configuration loaded
```

下面我们要设置redis开机启动

```
chkconfig redis on 
```
执行上面命令过程中，可能会提示这样的信息``服务 redis 不支持 chkconfig``，在/etc/init.d/redis脚本``#!/bin/sh``添加如下：

```
#chkconfig: 2345 80 90
#description:auto_run
```

这样就完成了，redis的配置，可以``reboot``，试试我们的服务有没有自动启动。