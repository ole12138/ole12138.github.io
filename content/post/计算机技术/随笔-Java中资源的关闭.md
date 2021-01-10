---
title: "Java中资源的关闭"
date: 2021-01-10T22:52:50+08:00
draft: false
categories: 
- Java
series:
- 随笔
tags:
- Java
- 关闭资源
---

# Java中资源的关闭

在Java编程过程中，如果打开了外部资源（文件、数据库连接、网络连接等），必须在这些外部资源使用完毕后，手动关闭它们。因为外部资源不由JVM管理，无法享用JVM的垃圾回收机制，如果不在编程时确保在正确的时机关闭外部资源，就会导致外部资源泄露，紧接着就会出现文件被异常占用，数据库连接过多导致连接池溢出等诸多很严重的问题。

一般

- 传统的资源关闭方式
  为了确保外部资源一定要被关闭，通常关闭代码被写入finally代码块中，当然还必须注意到关闭资源时可能抛出的异常。（最后抛出的异常是关闭资源过程中又新抛出的异常）

- JDK7及其之后的资源关闭方式
  确实，在JDK7以前，Java没有自动关闭外部资源的语法特性，直到JDK7中新增了try-with-resource语法，才实现了这一功能。
  注意1：只有一个外部资源的句柄对象实现了`AutoCloseable`接口，JDK7中便可以利用try-with-resource语法更优雅的关闭资源，消除板式代码。
  注意2：try-with-resource时，如果对外部资源的处理和对外部资源的关闭均遭遇了异常，“关闭异常”将被抑制，“处理异常”将被抛出，但“关闭异常”并没有丢失，而是存放在“处理异常”的被抑制的异常列表中。（最后抛出的异常是之前最先抛出的异常）

- lombok 方式
  使用 lombok 提供的注解 @Cleanup。

# 进一步阅读

[Java 之优雅地关闭资源 try-with-resource、lombok](https://blog.csdn.net/const_/article/details/93066769)

[Lombok之@Cleanup使用](https://blog.csdn.net/cauchy6317/article/details/102838510)

[使用Try-with-resources自动关闭资源](https://blog.csdn.net/wtopps/article/details/71108342)

[Jdk6 7 9 流关闭的新姿势](https://blog.csdn.net/ManagementAndJava/article/details/82910240?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.control&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.control)