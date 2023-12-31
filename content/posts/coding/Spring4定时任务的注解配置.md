---
title: "Spring4定时任务的注解配置"
date: 2014-11-03T22:57:00+08:00
draft: false
description: Spring4框架下，不使用xml配置文件，而采用注解配置定时任务。
tags: [Java]
slug: spring-scheduling-annotation-support
---

不想写繁琐的xml文件，所以试着用Spring的注解来配置定时任务。按网上好多写的那样，使用`@Scheduled`注解：

```java
@Scheduled(fixedRate=5000)
public void doSomething() {
    // something that should execute periodically
}
```

但是怎么也无法让定时任务跑起来。最后无奈，只有去[官网](http://docs.spring.io/spring/docs/current/spring-framework-reference/html/scheduling.html#scheduling-annotation-support)看英语了。

原来，还需要有一个允许定时任务的注解，放在一个有`@configuration`注解类中。

```java
@Configuration
@EnableAsync
@EnableScheduling
public class AppConfig {
}
```

其中，`@EnableAsysnc`是允许异步任务。至于`@Configuration`的说明，见[Spring的最新文档](http://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/context/annotation/Configuration.html)。
