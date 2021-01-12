---
title: "HTTPS简单介绍（转载）"
date: 2021-01-12T12:06:18+08:00
draft: false
categories: 
- HTTP
series:
- HTTP
- HTTPS
- java发起http请求
tags:
- HTTP
- HTTPS

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

# HTTPS加密机制

[SSL或TLS协议运行机制的概述（流程）](../SSL或TLS协议运行机制的概述（转载）)

[彻底搞懂HTTPS的加密机制（原理）](../彻底搞懂HTTPS的加密机制（转载）)

# 发起HTTPS请求

通过了解HTTPS加密机制，我们知道：

网站在使用HTTPS前的必要准备：

- 网站拥有用于非对称加密的公钥A、私钥A
- 网站向“**CA机构**”申请颁发一份**数字证书**以及对应的**数字签名**。
  数字证书里有（证书持有者、网站的公钥A以及其他信息）。
  数字签名是用**CA机构私钥**加密过的网站数字证书的hash值。

浏览器在发起HTTPS请求的过程中，在握手阶段(3次握手）：

- 浏览器发起请求(=>)
- 网站把证书和签名传输给浏览器。 (<=)
  我们可以认为，“浏览器保有**CA机构的公钥**”。（实际上是操作系统和浏览器预装了一些根证书，利用这些证书进一步获取CA机构的证书以及公钥）
  浏览器用CA机构的公钥验证网站的证书：
  - 浏览器用CA机构的公钥解密网站发来数字签名
  - 浏览器计算网站发来的数字证书的hash值
  - 比较签名与hash值是否相同
- 网站证书验证通过，即可从证书中取出证书持有者的公钥（网站的公钥）。
  浏览器随机生成一个用于对称加密的密钥X，用公钥A加密后传给服务器。(=>)
- 服务器拿到后用私钥A解密得到密钥X。这样双方就都拥有密钥X了，且别人无法知道它。之后双方所有数据都用密钥X加密解密。



除了使用浏览器，我们还可以使用其他客户端，比如Java就提供了HttpsURLConnection, 以及apache和JDK11的HttpClient等，也可以发起https请求。
与使用浏览器相同，https连接握手的过程中，网站也会发来证书和签名，也需要对证书进行验证。

就按使用证书的不同方式，分为3种情况：
1，信任系统提供的证书（信任权威CA颁发的证书）；
2，全部信任证书；
3，信任指定证书；

之后，在使用java发起http/https请求的博文中，会进行举例说明。

# 网站使用HTTPS

//TODO