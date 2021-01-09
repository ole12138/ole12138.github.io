---
title: "Java发起HTTP请求的几种方法"
date: 2021-01-07T16:31:40+08:00
draft: false
categories: 
- Java
- HTTP
series:
- Java实现HTTP请求
tags:
- Java
- HTTP
- Java实现HTTP请求
---

作为一个Java后端，需要通过HTTP请求其他的网络资源可以说是一个比较常见的case了；一般怎么做呢？

可能大部分的小伙伴直接捞起Apache的HttpClient开始做，

或者用其他的一些知名的开源库如OkHttp, 

当然原生的HttpURLConnection也是没问题的,

在Spring的生态下，则可以利用`RestTemplate`来发起Http请求。

