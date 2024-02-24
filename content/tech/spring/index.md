---
title: "Spring笔记"
description: 
date: 2021-12-15T20:21:58+08:00
image: 
math: 
license: 
hidden: false
comments: true
draft: false
tags:
categories:
  - 笔记
---

## [Spring_基础概念][1]
 - 基础概念
 - IOC、DI入门案例
 - Bean的配置 如何通过Factory配置Bean?(4.2.4、4.2.5)
 - DI相关内容
  - 如何通过setter、构造器进行依赖注入
  - 集合注入


## [Spring_day02][2]
 - 管理第三方bean(Druid)
 - 加载properties文件,增加命名空间？四种不同方式来加载`<context:property-placeholder location="class*:*.properties"/>`
 - 核心容器：不同的创建容器方式？
 - 在DI中，如何分别通过构造器（constructer-arg）或setter来注入属性或对象
 - 纯注解开发：
  - **配置**：使用Config类代替XML文件，之前使用`ApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");`,使用配置类后使用`ApplicationContext ctx = new AnnotationConfigApplicationContext(SpringConfig.class);`获取Spring上下文。
  - **Bean配置** 知识点3.3，@Scope @PostConstruct @PreDestroy
  - **DI依赖注入** 知识点：自动装配, 简单类型装配， @Value("${变量名}")简单类型装配， @PropertySource加载property文件（不支持通配符）
 - Spring整合MyBatis(§6.1)
 - Spring整合Junit(§6.2)
## [Spring AOP和事务处理][3]
 - 概念：连接点、切入点、通知、通知类、切面
 - §4 AOP切入点表达式、§6事务管理