---
title: "HTTPS简单介绍（转载）"
date: 2021-01-12T12:06:18+08:00
draft: false
categories: 
tags:
markup: pandoc
math: true
---

# HTTPS简单介绍（转载）

本章节非原创，转载来源：https://juejin.cn/post/6844903599303032845

### 一、HTTP存在的问题

#### 1.1 可能被窃听

1. HTTP 本身不具备加密的功能,HTTP 报文使用明文方式发送
2. 由于互联网是由联通世界各个地方的网络设施组成,所有发送和接收经过某些设备的数据都可能被截获或窥视。(例如大家都熟悉的抓包工具:Wireshark)

#### 1.2 认证问题

1. 无法确认你发送到的服务器就是真正的目标服务器(可能服务器是伪装的)
2. 无法确定返回的客户端是否是按照真实意图接收的客户端(可能是伪装的客户端)
3. 无法确定正在通信的对方是否具备访问权限，Web 服务器上某些重要的信息，只想发给特定用户即使是无意义的请求也会照单全收。无法阻止海量请求下的 DoS 攻击（Denial of Service，拒绝服务攻击）。

#### 1.3 可能被篡改

1.请求或响应在传输途中，遭攻击者拦截并篡改内容的攻击被称为中间人攻击（Man-in-the-Middle attack，MITM）。

### 二、HTTPS介绍

#### 2.1 什么是HTTPS

> 超文本传输安全协议（英语：Hypertext Transfer Protocol Secure，缩写：HTTPS，常称为HTTP over TLS，HTTP over SSL或HTTP Secure）是一种通过计算机网络进行安全通信的传输协议。HTTPS经由HTTP进行通信，但利用SSL/TLS来加密数据包。HTTPS开发的主要目的，是提供对网站服务器的身份认证，保护交换数据的隐私与完整性。

#### 2.2 HTTPS怎么解决上述问题

HTTPS是在通信接口部分用 TLS(Transport Layer Security 传输层安全性协议)，TLS协议采用主从式架构模型，用于在两个应用程序间通过网络创建起安全的连接，防止在交换数据时受到窃听及篡改。



![https和https](https://user-gold-cdn.xitu.io/2018/4/16/162cd3df803538f6?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



#### 2.3 SSL和TLS的关系

1. 传输层安全性协议（英语：Transport Layer Security，缩写作 TLS），及其前身安全套接层（Secure Sockets Layer，缩写作 SSL）是一种安全协议，目的是为互联网通信，提供安全及数据完整性保障。
2. 网景公司（Netscape）在1994年推出首版网页浏览器，网景导航者时，推出HTTPS协议，以SSL进行加密，这是SSL的起源。
3. IETF将SSL进行标准化，1999年公布第一版TLS标准文件。随后又公布RFC 5246 （2008年8月）与 RFC 6176 （2011年3月）。在浏览器、电子邮件、即时通信、VoIP、网络传真等应用程序中，广泛支持这个协议。

#### 2.4 TLS/SSL 协议

> HTTPS 协议的主要功能基本都依赖于 TLS/SSL 协议，TLS/SSL 的功能实现主要依赖于三类基本算法：`散列函数` 、`对称加密`和`非对称加密`，其利用非对称加密实现身份认证和密钥协商，对称加密算法采用协商的密钥对数据加密，基于散列函数验证信息的完整性。
>
> ![TLS/SSL协议](https://user-gold-cdn.xitu.io/2018/4/20/162e3583d5cb5b1e?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

> SSL/TLS协议运行机制 可以看阮老师的[SSL/TLS协议运行机制的概述](http://ruanyifeng.com/blog/2014/02/ssl_tls.html)

> RSA加密算法原理可以看阮老师的这两篇文章 [RSA算法原理（一）](http://www.ruanyifeng.com/blog/2013/06/rsa_algorithm_part_one.html)、 [RSA算法原理（二）](http://www.ruanyifeng.com/blog/2013/07/rsa_algorithm_part_two.html)


作者：Keely40285
链接：https://juejin.cn/post/6844903599303032845
来源：掘金
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。