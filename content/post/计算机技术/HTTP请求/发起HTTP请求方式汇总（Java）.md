---
title: "Java发起HTTP请求方式汇总"
date: 2021-01-07T16:31:40+08:00
draft: false
categories: 
- Java
- HTTP
series:
- Java发起HTTP请求
tags:
- Java
- HTTP
- Java发起HTTP请求
---

作为一个Java后端，需要通过HTTP请求其他的网络资源可以说是一个比较常见的case了；一般怎么做呢？

- 直接捞起Apache的HttpClient开始做 (只支持到HTTP/1.1,最近的Http5.1-beta开始支持HTTP/2)
- 知名的开源库如OkHttp
- 原生的HttpURLConnection （Since: JDK1.1）
- 原生的HttpClient （JDK11）
- Spring的生态中可以利用`RestTemplate`来发起Http请求。

发送HTTP请求时，要注意区分http和https类型的请求

