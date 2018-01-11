````---
title: spring cloud config server启动异常
layout: post
author: 王召辉
catalog: true
tags: 
 - spring cloud
---

上篇文章讲到了，使用``spring-cloud``两种maven配置方式。基于想统一管理依赖版本的目的，所以项目中一般采用了第二种方式，继承我们自己的pom文件。先看下项目的pom文件

```
<parent>
    <groupId>cn.xxxxx</groupId>
    <artifactId>xxxxx-parent-pom</artifactId>
    <version>1.0-SNAPSHOT</version>
</parent>

<groupId>cn.xxxxx</groupId>
<artifactId>xxxxx-config-center</artifactId>
<version>1.0.0</version>


<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-config-server</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-eureka</artifactId>
    </dependency>
</dependencies>
```

关于parent pom的配置如下

```
<properties>
    <spring.version>4.3.10.RELEASE</spring.version>
    <spring-boot.version>1.5.9.RELEASE</spring-boot.version>
</properties>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-dependencies</artifactId>
    <version>${spring-boot.version}</version>
    <type>pom</type>
    <scope>import</scope>
</dependency>

<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-dependencies</artifactId>
    <version>${spring-cloud.version}</version>
    <type>pom</type>
    <scope>import</scope>
</dependency>
```

一切都很合理，但是启动后报错了，一坨错误信息如下:

``` java
org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'org.springframework.boot.actuate.autoconfigure.EndpointAutoConfiguration': Bean instantiation via constructor failed; nested exception is org.springframework.beans.BeanInstantiationException: Failed to instantiate [org.springframework.boot.actuate.autoconfigure.EndpointAutoConfiguration$$EnhancerBySpringCGLIB$$bbfa9d76]: Constructor threw exception; nested exception is org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'configServerHealthIndicator' defined in class path resource [org/springframework/cloud/config/server/config/EnvironmentRepositoryConfiguration.class]: Unsatisfied dependency expressed through method 'configServerHealthIndicator' parameter 0; nested exception is org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'org.springframework.cloud.config.server.config.CompositeConfiguration': Unsatisfied dependency expressed through method 'setEnvironmentRepos' parameter 0; nested exception is org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'org.springframework.cloud.config.server.config.DefaultRepositoryConfiguration': Unsatisfied dependency expressed through field 'transportConfigCallback'; nested exception is org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'propertiesBasedSshTransportCallback' defined in class path resource [org/springframework/cloud/config/server/config/TransportConfiguration.class]: Bean instantiation via factory method failed; nested exception is org.springframework.beans.BeanInstantiationException: Failed to instantiate [org.eclipse.jgit.api.TransportConfigCallback]: Factory method 'propertiesBasedSshTransportCallback' threw exception; nested exception is java.lang.AbstractMethodError: org.springframework.validation.beanvalidation.LocalValidatorFactoryBean.forExecutables()Ljavax/validation/executable/ExecutableValidator;
	at org.springframework.beans.factory.support.ConstructorResolver.autowireConstructor(ConstructorResolver.java:279) ~[spring-beans-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.autowireConstructor(AbstractAutowireCapableBeanFactory.java:1193) ~[spring-beans-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBeanInstance(AbstractAutowireCapableBeanFactory.java:1095) ~[spring-beans-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.doCreateBean(AbstractAutowireCapableBeanFactory.java:513) ~[spring-beans-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBean(AbstractAutowireCapableBeanFactory.java:483) ~[spring-beans-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.beans.factory.support.AbstractBeanFactory$1.getObject(AbstractBeanFactory.java:306) ~[spring-beans-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.getSingleton(DefaultSingletonBeanRegistry.java:230) ~[spring-beans-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.beans.factory.support.AbstractBeanFactory.doGetBean(AbstractBeanFactory.java:302) ~[spring-beans-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.beans.factory.support.AbstractBeanFactory.getBean(AbstractBeanFactory.java:197) ~[spring-beans-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.beans.factory.support.DefaultListableBeanFactory.preInstantiateSingletons(DefaultListableBeanFactory.java:761) ~[spring-beans-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.context.support.AbstractApplicationContext.finishBeanFactoryInitialization(AbstractApplicationContext.java:867) ~[spring-context-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.context.support.AbstractApplicationContext.refresh(AbstractApplicationContext.java:543) ~[spring-context-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.boot.context.embedded.EmbeddedWebApplicationContext.refresh(EmbeddedWebApplicationContext.java:122) ~[spring-boot-1.5.9.RELEASE.jar:1.5.9.RELEASE]
	at org.springframework.boot.SpringApplication.refresh(SpringApplication.java:693) [spring-boot-1.5.9.RELEASE.jar:1.5.9.RELEASE]
	at org.springframework.boot.SpringApplication.refreshContext(SpringApplication.java:360) [spring-boot-1.5.9.RELEASE.jar:1.5.9.RELEASE]
	at org.springframework.boot.SpringApplication.run(SpringApplication.java:303) [spring-boot-1.5.9.RELEASE.jar:1.5.9.RELEASE]
	at org.springframework.boot.SpringApplication.run(SpringApplication.java:1118) [spring-boot-1.5.9.RELEASE.jar:1.5.9.RELEASE]
	at org.springframework.boot.SpringApplication.run(SpringApplication.java:1107) [spring-boot-1.5.9.RELEASE.jar:1.5.9.RELEASE]
	at cn.xxxxx.config.center.Launcher.main(Launcher.java:22) [classes/:na]
Caused by: org.springframework.beans.BeanInstantiationException: Failed to instantiate [org.springframework.boot.actuate.autoconfigure.EndpointAutoConfiguration$$EnhancerBySpringCGLIB$$bbfa9d76]: Constructor threw exception; nested exception is org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'configServerHealthIndicator' defined in class path resource [org/springframework/cloud/config/server/config/EnvironmentRepositoryConfiguration.class]: Unsatisfied dependency expressed through method 'configServerHealthIndicator' parameter 0; nested exception is org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'org.springframework.cloud.config.server.config.CompositeConfiguration': Unsatisfied dependency expressed through method 'setEnvironmentRepos' parameter 0; nested exception is org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'org.springframework.cloud.config.server.config.DefaultRepositoryConfiguration': Unsatisfied dependency expressed through field 'transportConfigCallback'; nested exception is org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'propertiesBasedSshTransportCallback' defined in class path resource [org/springframework/cloud/config/server/config/TransportConfiguration.class]: Bean instantiation via factory method failed; nested exception is org.springframework.beans.BeanInstantiationException: Failed to instantiate [org.eclipse.jgit.api.TransportConfigCallback]: Factory method 'propertiesBasedSshTransportCallback' threw exception; nested exception is java.lang.AbstractMethodError: org.springframework.validation.beanvalidation.LocalValidatorFactoryBean.forExecutables()Ljavax/validation/executable/ExecutableValidator;
	at org.springframework.beans.BeanUtils.instantiateClass(BeanUtils.java:154) ~[spring-beans-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.beans.factory.support.SimpleInstantiationStrategy.instantiate(SimpleInstantiationStrategy.java:122) ~[spring-beans-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.beans.factory.support.ConstructorResolver.autowireConstructor(ConstructorResolver.java:271) ~[spring-beans-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	... 18 common frames omitted
Caused by: org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'configServerHealthIndicator' defined in class path resource [org/springframework/cloud/config/server/config/EnvironmentRepositoryConfiguration.class]: Unsatisfied dependency expressed through method 'configServerHealthIndicator' parameter 0; nested exception is org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'org.springframework.cloud.config.server.config.CompositeConfiguration': Unsatisfied dependency expressed through method 'setEnvironmentRepos' parameter 0; nested exception is org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'org.springframework.cloud.config.server.config.DefaultRepositoryConfiguration': Unsatisfied dependency expressed through field 'transportConfigCallback'; nested exception is org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'propertiesBasedSshTransportCallback' defined in class path resource [org/springframework/cloud/config/server/config/TransportConfiguration.class]: Bean instantiation via factory method failed; nested exception is org.springframework.beans.BeanInstantiationException: Failed to instantiate [org.eclipse.jgit.api.TransportConfigCallback]: Factory method 'propertiesBasedSshTransportCallback' threw exception; nested exception is java.lang.AbstractMethodError: org.springframework.validation.beanvalidation.LocalValidatorFactoryBean.forExecutables()Ljavax/validation/executable/ExecutableValidator;
	at org.springframework.beans.factory.support.ConstructorResolver.createArgumentArray(ConstructorResolver.java:749) ~[spring-beans-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.beans.factory.support.ConstructorResolver.instantiateUsingFactoryMethod(ConstructorResolver.java:467) ~[spring-beans-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.instantiateUsingFactoryMethod(AbstractAutowireCapableBeanFactory.java:1173) ~[spring-beans-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBeanInstance(AbstractAutowireCapableBeanFactory.java:1067) ~[spring-beans-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.doCreateBean(AbstractAutowireCapableBeanFactory.java:513) ~[spring-beans-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBean(AbstractAutowireCapableBeanFactory.java:483) ~[spring-beans-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.beans.factory.support.AbstractBeanFactory$1.getObject(AbstractBeanFactory.java:306) ~[spring-beans-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.getSingleton(DefaultSingletonBeanRegistry.java:230) ~[spring-beans-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.beans.factory.support.AbstractBeanFactory.doGetBean(AbstractBeanFactory.java:302) ~[spring-beans-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.beans.factory.support.AbstractBeanFactory.getBean(AbstractBeanFactory.java:202) ~[spring-beans-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.beans.factory.config.DependencyDescriptor.resolveCandidate(DependencyDescriptor.java:208) ~[spring-beans-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.beans.factory.support.DefaultListableBeanFactory.addCandidateEntry(DefaultListableBeanFactory.java:1316) ~[spring-beans-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.beans.factory.support.DefaultListableBeanFactory.findAutowireCandidates(DefaultListableBeanFactory.java:1282) ~[spring-beans-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.beans.factory.support.DefaultListableBeanFactory.resolveMultipleBeans(DefaultListableBeanFactory.java:1205) ~[spring-beans-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.beans.factory.support.DefaultListableBeanFactory.doResolveDependency(DefaultListableBeanFactory.java:1096) ~[spring-beans-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.beans.factory.support.DefaultListableBeanFactory$DependencyObjectProvider.getIfAvailable(DefaultListableBeanFactory.java:1662) ~[spring-beans-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.boot.actuate.autoconfigure.EndpointAutoConfiguration.<init>(EndpointAutoConfiguration.java:102) ~[spring-boot-actuator-1.5.9.RELEASE.jar:1.5.9.RELEASE]
	at org.springframework.boot.actuate.autoconfigure.EndpointAutoConfiguration$$EnhancerBySpringCGLIB$$bbfa9d76.<init>(<generated>) ~[spring-boot-actuator-1.5.9.RELEASE.jar:1.5.9.RELEASE]
	at sun.reflect.NativeConstructorAccessorImpl.newInstance0(Native Method) ~[na:1.8.0_77]
	at sun.reflect.NativeConstructorAccessorImpl.newInstance(NativeConstructorAccessorImpl.java:62) ~[na:1.8.0_77]
	at sun.reflect.DelegatingConstructorAccessorImpl.newInstance(DelegatingConstructorAccessorImpl.java:45) ~[na:1.8.0_77]
	at java.lang.reflect.Constructor.newInstance(Constructor.java:423) ~[na:1.8.0_77]
	at org.springframework.beans.BeanUtils.instantiateClass(BeanUtils.java:142) ~[spring-beans-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	... 20 common frames omitted
Caused by: org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'org.springframework.cloud.config.server.config.CompositeConfiguration': Unsatisfied dependency expressed through method 'setEnvironmentRepos' parameter 0; nested exception is org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'org.springframework.cloud.config.server.config.DefaultRepositoryConfiguration': Unsatisfied dependency expressed through field 'transportConfigCallback'; nested exception is org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'propertiesBasedSshTransportCallback' defined in class path resource [org/springframework/cloud/config/server/config/TransportConfiguration.class]: Bean instantiation via factory method failed; nested exception is org.springframework.beans.BeanInstantiationException: Failed to instantiate [org.eclipse.jgit.api.TransportConfigCallback]: Factory method 'propertiesBasedSshTransportCallback' threw exception; nested exception is java.lang.AbstractMethodError: org.springframework.validation.beanvalidation.LocalValidatorFactoryBean.forExecutables()Ljavax/validation/executable/ExecutableValidator;
	at org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor$AutowiredMethodElement.inject(AutowiredAnnotationBeanPostProcessor.java:667) ~[spring-beans-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.beans.factory.annotation.InjectionMetadata.inject(InjectionMetadata.java:88) ~[spring-beans-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor.postProcessPropertyValues(AutowiredAnnotationBeanPostProcessor.java:366) ~[spring-beans-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.populateBean(AbstractAutowireCapableBeanFactory.java:1264) ~[spring-beans-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.doCreateBean(AbstractAutowireCapableBeanFactory.java:553) ~[spring-beans-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBean(AbstractAutowireCapableBeanFactory.java:483) ~[spring-beans-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.beans.factory.support.AbstractBeanFactory$1.getObject(AbstractBeanFactory.java:306) ~[spring-beans-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.getSingleton(DefaultSingletonBeanRegistry.java:230) ~[spring-beans-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.beans.factory.support.AbstractBeanFactory.doGetBean(AbstractBeanFactory.java:302) ~[spring-beans-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.beans.factory.support.AbstractBeanFactory.getBean(AbstractBeanFactory.java:197) ~[spring-beans-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.beans.factory.support.ConstructorResolver.instantiateUsingFactoryMethod(ConstructorResolver.java:372) ~[spring-beans-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.instantiateUsingFactoryMethod(AbstractAutowireCapableBeanFactory.java:1173) ~[spring-beans-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBeanInstance(AbstractAutowireCapableBeanFactory.java:1067) ~[spring-beans-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.doCreateBean(AbstractAutowireCapableBeanFactory.java:513) ~[spring-beans-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBean(AbstractAutowireCapableBeanFactory.java:483) ~[spring-beans-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.beans.factory.support.AbstractBeanFactory$1.getObject(AbstractBeanFactory.java:306) ~[spring-beans-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.getSingleton(DefaultSingletonBeanRegistry.java:230) ~[spring-beans-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.beans.factory.support.AbstractBeanFactory.doGetBean(AbstractBeanFactory.java:302) ~[spring-beans-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.beans.factory.support.AbstractBeanFactory.getBean(AbstractBeanFactory.java:202) ~[spring-beans-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.beans.factory.config.DependencyDescriptor.resolveCandidate(DependencyDescriptor.java:208) ~[spring-beans-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.beans.factory.support.DefaultListableBeanFactory.doResolveDependency(DefaultListableBeanFactory.java:1138) ~[spring-beans-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.beans.factory.support.DefaultListableBeanFactory.resolveDependency(DefaultListableBeanFactory.java:1066) ~[spring-beans-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.beans.factory.support.ConstructorResolver.resolveAutowiredArgument(ConstructorResolver.java:835) ~[spring-beans-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.beans.factory.support.ConstructorResolver.createArgumentArray(ConstructorResolver.java:741) ~[spring-beans-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	... 42 common frames omitted
Caused by: org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'org.springframework.cloud.config.server.config.DefaultRepositoryConfiguration': Unsatisfied dependency expressed through field 'transportConfigCallback'; nested exception is org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'propertiesBasedSshTransportCallback' defined in class path resource [org/springframework/cloud/config/server/config/TransportConfiguration.class]: Bean instantiation via factory method failed; nested exception is org.springframework.beans.BeanInstantiationException: Failed to instantiate [org.eclipse.jgit.api.TransportConfigCallback]: Factory method 'propertiesBasedSshTransportCallback' threw exception; nested exception is java.lang.AbstractMethodError: org.springframework.validation.beanvalidation.LocalValidatorFactoryBean.forExecutables()Ljavax/validation/executable/ExecutableValidator;
	at org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor$AutowiredFieldElement.inject(AutowiredAnnotationBeanPostProcessor.java:588) ~[spring-beans-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.beans.factory.annotation.InjectionMetadata.inject(InjectionMetadata.java:88) ~[spring-beans-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor.postProcessPropertyValues(AutowiredAnnotationBeanPostProcessor.java:366) ~[spring-beans-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.populateBean(AbstractAutowireCapableBeanFactory.java:1264) ~[spring-beans-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.doCreateBean(AbstractAutowireCapableBeanFactory.java:553) ~[spring-beans-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBean(AbstractAutowireCapableBeanFactory.java:483) ~[spring-beans-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.beans.factory.support.AbstractBeanFactory$1.getObject(AbstractBeanFactory.java:306) ~[spring-beans-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.getSingleton(DefaultSingletonBeanRegistry.java:230) ~[spring-beans-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.beans.factory.support.AbstractBeanFactory.doGetBean(AbstractBeanFactory.java:302) ~[spring-beans-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.beans.factory.support.AbstractBeanFactory.getBean(AbstractBeanFactory.java:197) ~[spring-beans-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.beans.factory.support.ConstructorResolver.instantiateUsingFactoryMethod(ConstructorResolver.java:372) ~[spring-beans-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.instantiateUsingFactoryMethod(AbstractAutowireCapableBeanFactory.java:1173) ~[spring-beans-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBeanInstance(AbstractAutowireCapableBeanFactory.java:1067) ~[spring-beans-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.doCreateBean(AbstractAutowireCapableBeanFactory.java:513) ~[spring-beans-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBean(AbstractAutowireCapableBeanFactory.java:483) ~[spring-beans-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.beans.factory.support.AbstractBeanFactory$1.getObject(AbstractBeanFactory.java:306) ~[spring-beans-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.getSingleton(DefaultSingletonBeanRegistry.java:230) ~[spring-beans-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.beans.factory.support.AbstractBeanFactory.doGetBean(AbstractBeanFactory.java:302) ~[spring-beans-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.beans.factory.support.AbstractBeanFactory.getBean(AbstractBeanFactory.java:202) ~[spring-beans-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.beans.factory.config.DependencyDescriptor.resolveCandidate(DependencyDescriptor.java:208) ~[spring-beans-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.beans.factory.support.DefaultListableBeanFactory.addCandidateEntry(DefaultListableBeanFactory.java:1316) ~[spring-beans-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.beans.factory.support.DefaultListableBeanFactory.findAutowireCandidates(DefaultListableBeanFactory.java:1282) ~[spring-beans-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.beans.factory.support.DefaultListableBeanFactory.resolveMultipleBeans(DefaultListableBeanFactory.java:1180) ~[spring-beans-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.beans.factory.support.DefaultListableBeanFactory.doResolveDependency(DefaultListableBeanFactory.java:1096) ~[spring-beans-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.beans.factory.support.DefaultListableBeanFactory.resolveDependency(DefaultListableBeanFactory.java:1066) ~[spring-beans-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor$AutowiredMethodElement.inject(AutowiredAnnotationBeanPostProcessor.java:659) ~[spring-beans-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	... 65 common frames omitted
Caused by: org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'propertiesBasedSshTransportCallback' defined in class path resource [org/springframework/cloud/config/server/config/TransportConfiguration.class]: Bean instantiation via factory method failed; nested exception is org.springframework.beans.BeanInstantiationException: Failed to instantiate [org.eclipse.jgit.api.TransportConfigCallback]: Factory method 'propertiesBasedSshTransportCallback' threw exception; nested exception is java.lang.AbstractMethodError: org.springframework.validation.beanvalidation.LocalValidatorFactoryBean.forExecutables()Ljavax/validation/executable/ExecutableValidator;
	at org.springframework.beans.factory.support.ConstructorResolver.instantiateUsingFactoryMethod(ConstructorResolver.java:599) ~[spring-beans-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.instantiateUsingFactoryMethod(AbstractAutowireCapableBeanFactory.java:1173) ~[spring-beans-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBeanInstance(AbstractAutowireCapableBeanFactory.java:1067) ~[spring-beans-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.doCreateBean(AbstractAutowireCapableBeanFactory.java:513) ~[spring-beans-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBean(AbstractAutowireCapableBeanFactory.java:483) ~[spring-beans-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.beans.factory.support.AbstractBeanFactory$1.getObject(AbstractBeanFactory.java:306) ~[spring-beans-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.getSingleton(DefaultSingletonBeanRegistry.java:230) ~[spring-beans-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.beans.factory.support.AbstractBeanFactory.doGetBean(AbstractBeanFactory.java:302) ~[spring-beans-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.beans.factory.support.AbstractBeanFactory.getBean(AbstractBeanFactory.java:202) ~[spring-beans-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.beans.factory.config.DependencyDescriptor.resolveCandidate(DependencyDescriptor.java:208) ~[spring-beans-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.beans.factory.support.DefaultListableBeanFactory.doResolveDependency(DefaultListableBeanFactory.java:1138) ~[spring-beans-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.beans.factory.support.DefaultListableBeanFactory.resolveDependency(DefaultListableBeanFactory.java:1066) ~[spring-beans-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor$AutowiredFieldElement.inject(AutowiredAnnotationBeanPostProcessor.java:585) ~[spring-beans-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	... 90 common frames omitted
Caused by: org.springframework.beans.BeanInstantiationException: Failed to instantiate [org.eclipse.jgit.api.TransportConfigCallback]: Factory method 'propertiesBasedSshTransportCallback' threw exception; nested exception is java.lang.AbstractMethodError: org.springframework.validation.beanvalidation.LocalValidatorFactoryBean.forExecutables()Ljavax/validation/executable/ExecutableValidator;
	at org.springframework.beans.factory.support.SimpleInstantiationStrategy.instantiate(SimpleInstantiationStrategy.java:189) ~[spring-beans-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.beans.factory.support.ConstructorResolver.instantiateUsingFactoryMethod(ConstructorResolver.java:588) ~[spring-beans-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	... 102 common frames omitted
Caused by: java.lang.AbstractMethodError: org.springframework.validation.beanvalidation.LocalValidatorFactoryBean.forExecutables()Ljavax/validation/executable/ExecutableValidator;
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method) ~[na:1.8.0_77]
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62) ~[na:1.8.0_77]
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43) ~[na:1.8.0_77]
	at java.lang.reflect.Method.invoke(Method.java:498) ~[na:1.8.0_77]
	at org.springframework.aop.support.AopUtils.invokeJoinpointUsingReflection(AopUtils.java:333) ~[spring-aop-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.aop.framework.JdkDynamicAopProxy.invoke(JdkDynamicAopProxy.java:207) ~[spring-aop-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at com.sun.proxy.$Proxy65.forExecutables(Unknown Source) ~[na:na]
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method) ~[na:1.8.0_77]
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62) ~[na:1.8.0_77]
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43) ~[na:1.8.0_77]
	at java.lang.reflect.Method.invoke(Method.java:498) ~[na:1.8.0_77]
	at org.springframework.util.ReflectionUtils.invokeMethod(ReflectionUtils.java:216) ~[spring-core-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.util.ReflectionUtils.invokeMethod(ReflectionUtils.java:201) ~[spring-core-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.validation.beanvalidation.MethodValidationInterceptor.invoke(MethodValidationInterceptor.java:119) ~[spring-context-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:179) ~[spring-aop-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.aop.framework.CglibAopProxy$DynamicAdvisedInterceptor.intercept(CglibAopProxy.java:673) ~[spring-aop-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.cloud.config.server.ssh.SshUriProperties$$EnhancerBySpringCGLIB$$b6e0d00a.isIgnoreLocalSshSettings(<generated>) ~[spring-cloud-config-server-1.4.0.RELEASE.jar:1.4.0.RELEASE]
	at org.springframework.cloud.config.server.config.TransportConfiguration.propertiesBasedSshTransportCallback(TransportConfiguration.java:43) ~[spring-cloud-config-server-1.4.0.RELEASE.jar:1.4.0.RELEASE]
	at org.springframework.cloud.config.server.config.TransportConfiguration$$EnhancerBySpringCGLIB$$64d54d69.CGLIB$propertiesBasedSshTransportCallback$0(<generated>) ~[spring-cloud-config-server-1.4.0.RELEASE.jar:1.4.0.RELEASE]
	at org.springframework.cloud.config.server.config.TransportConfiguration$$EnhancerBySpringCGLIB$$64d54d69$$FastClassBySpringCGLIB$$3ce44781.invoke(<generated>) ~[spring-cloud-config-server-1.4.0.RELEASE.jar:1.4.0.RELEASE]
	at org.springframework.cglib.proxy.MethodProxy.invokeSuper(MethodProxy.java:228) ~[spring-core-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.context.annotation.ConfigurationClassEnhancer$BeanMethodInterceptor.intercept(ConfigurationClassEnhancer.java:358) ~[spring-context-4.3.10.RELEASE.jar:4.3.10.RELEASE]
	at org.springframework.cloud.config.server.config.TransportConfiguration$$EnhancerBySpringCGLIB$$64d54d69.propertiesBasedSshTransportCallback(<generated>) ~[spring-cloud-config-server-1.4.0.RELEASE.jar:1.4.0.RELEASE]
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method) ~[na:1.8.0_77]
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62) ~[na:1.8.0_77]
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43) ~[na:1.8.0_77]
	at java.lang.reflect.Method.invoke(Method.java:498) ~[na:1.8.0_77]
	at org.springframework.beans.factory.support.SimpleInstantiationStrategy.instantiate(SimpleInstantiationStrategy.java:162) ~[spring-beans-4.3.10.RELEASE.jar:4.3.10.RELEASE]
```

完全不明所以，于是各种跟进源码，看启动流程哪里不对，因为我注意到这种方式跟继承spring-boot的pom引入的jar是相同的。但是没找到对应的思路。。。

就这样了连续搞了两个晚上还是不行。。。。

于是干脆进行使用``mvn -X dependency:tree``比对使用两种形式的依赖的包有什么不一样。但是所有的jar包都一样。。。

除了，spring的版本号

随干脆把spring的版本号改成一样。。。。

WTF

竟然启动成功了。。。````