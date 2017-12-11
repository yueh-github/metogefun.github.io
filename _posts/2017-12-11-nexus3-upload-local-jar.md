---
title: nexus 3.x 上传本地jar
layout: post
author: 王召辉
catalog: true
tags: 
 - nexus
---

如果想要上传非maven管理的jar到nexus 3，语法如下：

```
mvn deploy:deploy-file 
-DgroupId=<group-id>\
-DartifactId=<artifact-id>\
-Dversion=<version>\
-Dpackaging=<type-of-packaging>\
-Dfile=<path-to-file>\
-DrepositoryId=<id-to-map-on-server-section-of-settings.xml>\
-Durl=<url-of-the-repository-to-deploy>
```

举一个简单的栗子：

```
mvn deploy:deploy-file 
    -DgroupId=lt.jave 
    -DartifactId=jave 
    -Dversion=1.0.2 
    -Dfile=jave-1.0.2.jar 
    -DrepositoryId=nexus 
    -Durl=http://localhost:8081/repository/maven-releases/
```

DrepositoryId=nexus指的是setting配置的验证的id