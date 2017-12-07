---
title: tomcat 400 bad request
layout: post
author: 王召辉
catalog: true
tags: 
 - tomcat
---

今天在本机调试Java程序，突然看到客户端报``400 Bad Request``问题，赶紧查看后端log,异常如下

```
java.lang.IllegalArgumentException: Invalid character found in the request target. The valid characters are defined in RFC 7230 and RFC 3986
```

尝试各种解决方案未果，后在[stackoverflow](https://stackoverflow.com/)上找到对应的方案。

!(way)[{{ site.url }}/imgs/tomcat/400-bad-request.jpg]

果断降版本，继续测试