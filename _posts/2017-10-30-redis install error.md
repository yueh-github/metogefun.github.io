---
title: Redis异常汇总
layout: post
tags: 
 - redis
---

## zmalloc.h:50:31: 致命错误：jemalloc/jemalloc.h：没有那个文件或目录

在进入redis目录进行过make时，会出现该异常。解决方案：make MALLOC=libc
