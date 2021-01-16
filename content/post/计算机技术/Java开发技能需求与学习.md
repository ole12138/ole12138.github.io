---
title: "Java开发技能需求"
date: 2021-01-05T11:02:34+08:00
draft: true
categories: 
- 计算机技术
tags:
mermaid: true
---

# Java开发技能需求（待掌握技能）

参考：BOSS直聘招聘需求

常用Java中间件

dubbo，分布式缓存，消息队列

redis，kafka，zookeeper，etcd

大数据：Hadoop，spark，flink

Spring，SpringBoot，SpringCloud

分布式数据库：DRDS，oceanBase，ShardingSphere

docker，CI，CD工具

maven，gradle

Java多线程、缓存、安全

HTTP、Rest相关概念

VUE，REAT

# Java生态圈技术框架、中间件、系统架构汇总

参考：[Java生态圈技术框架、中间件、系统架构汇总](https://juejin.cn/post/6844903620979212296)

# Java开发技能学习

```mermaid
gantt
dateFormat  YYYY-MM-DD
title Java开发技能学习

section Spring
Spring项目的建立（xml）							:done, 	springInit, 2021-01-06, 2021-01-07
java发起Http请求-restTemplate						:done, restTemplate, 2021-01-07, 3d	
Spring之Interceptor								:active, crit,2021-01-10,2d
SpringMVC之Servlet与Interceptor					:2021-01-13,2d
Spring缓存										:2021-01-15,2d
Spring之redis以及消息中间件的使用					:2021-01-17,5d

section SpringBoot
SpringBoot项目的建立（xml）						:done, 	springBootInit, 2021-01-06, 2021-01-07
SpringBoot之restfulService						:done, 	restfulService, 2021-01-06, 2021-01-07

section Maven
Maven的pom的标签								:done, mavanPom, 2021-01-05, 2021-01-06

section Java
Java函数式编程与lambda表达式							:done, javaLambda, 2021-01-07, 2021-01-08
Java代理模式										:done, javaProxy, 2021-01-09, 1d
java发起Http请求-apache的HttpClient							:crit,httpClient,after restTemplate,1d
java发起Http请求-HttpUrlConnection					:done,2021-01-10,1d
java发起Http请求-OkHttp								:active,1d
```





