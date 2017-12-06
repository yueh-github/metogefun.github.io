---
title: Maven 依赖
layout: post
tags:
  - maven
---

maven的依赖范围可以通过__scope__来指定，在使用过程中很少指定它，一般使用默认，比较常见的就是Junit，它的作用域为test，在进行maven test中使用。maven作用域有哪些类型？什么传递性依赖？依赖传递范围是什么？如果pom文件中同一个依赖有多个版本会有什么问题？等等问题，我们会在下面一一进行详解。

## 依赖范围

在介绍依赖范围之前，首先我们先了解下maven中的classpath，maven实际的使用中，存在三种calsspapth：编译classpath、测试classpath、运行classpath。

依赖范围就是用来控制依赖与这三种classpath的关系。

| 依赖范围<br/>(scope)  | 对于编译<br />classpath有效  | 对于测试<br />classpath有效 |对于运行时<br/>classpath有效|例子|
|:------------- |:---------------:| -------------:|--------------:|----------:|
| compile<br/>(默认)      | Y | Y | Y| spring-core|
|test| — | Y| —| junit |
|provider | Y | Y | —| servlet-api|
|runtime| — | Y | Y| JDBC驱动|
|system| Y| Y | —| 本地Maven库之外的类库文件|

## 传递性依赖和依赖范围

什么是传递依赖，举个例子A依赖于B，B依赖于C，A依赖B属于第一直接依赖，B依赖C是第二直接依赖。A依赖C属于传递性依赖。下面第一列是是第一依赖范围，第一行为第二直接依赖

| 依赖范围  | compile | test | provider | runtime |
|:------------- |:----------:| -----:|------------:|----------:|
| compile      | compile |  —| —|runtime|
| test| test| — | — | test |
| provider | provider | — | provider | provider |
| runtime | runtime| — | — | runtime |


## 依赖调节

如 A -> B -> C -> X(1.0), A -> D -> X(2.0)。这样会有两个版本的X，为了避免造成重复依赖，Maven根据路径最短原则会选择X(2.0)。如果相同长度如，A -> C -> X(1.0)，A -> D ->X(2.0)，会根据在POM文件中声明的顺序来确定，X(1.0)在X(2.0)之前所以 1.0版本会被使用。


## 可选依赖

依赖声明时使用<optional>时，则声明了可选依赖。例如 A -> B， B -> X（可选）根据依赖的传递性，三个依赖的范围都为compiler，但是X是可选依赖，所以X对于A不会有任何影响。

## 排除依赖

依赖传递会给项目隐式地引入很多依赖，这个时候可能会带来问题。例如，当你使用spring-core时，他默认会引入common-logging。项目中可能你并不需要这个日志系统。可以使用<exclustion>来排除该依赖

``` xml
<dependency>
	<groupId>org.springframework</group>
	<artifactId>spring-core></artifactId>
	<version>5.0.0.RELEASE</version>
	<exclustions>
		<exclution>
			<groupId>commons-logging</groupId>
			<artifactId>commons-logging</artifactId>
		</exclution>
	</exclustions>
</denpendency>
```

## 依赖优化

可以使用``mvn denpendency:list``来查看项目中所有依赖。如果你想知道你第一直接依赖，以及后面的传递性依赖等兴义可以用过``mvn denpendency:tree``命令。你可以使用``mvn dependency:analyze``可以查看使用到的和未使用到的依赖信息。