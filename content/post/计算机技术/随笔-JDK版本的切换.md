---
title: "随笔-JDK版本的切换"
date: 2021-01-11T18:11:08+08:00
draft: false
categories: 
- Java
series:
- 随笔
tags:
- Java
- 关闭资源
- JDK版本的切换
---

# JDK版本的切换

有时为了使用新一点的API，项目需要使用高版本的JDK，比如JDK8切换到JDK11.

有时为了维护旧项目，需要使用旧版本的JDK，比如JDK8切换到JDK7.

这就需要安装和切换对应的JDK

## 安装对应版本的JDK

一般我们使用Java SE版本。Linux自带的是openJDK，windows下我们一般使用Oracle 的JDK。

下载安装过程略。

## IDEA中切换SDK

我们的项目一般都是在IDE工具中开发的。Java典型的IDE是intellij IDEA。

在IDEA中打开对应的项目。作如下操作：

> File->Project Structure->SDKs->点"+"号新建一个JDK，JDK所在目录选定前面JDK的安装目录。

这样，项目就会使用对应的JDK。

若已经按之前的JDK版本打开/导入了项目，为了使新的JDK生效，还需要做如下操作：

> File->Project Structure->Modules->点击右侧Dependencis选项卡->Module SDK选择刚刚新建的那个JDK

## Maven中切换compiler版本

我们一般是使用Maven构建程序的。

打开Maven要用到的pom文件，编译插件部分作如下修改：

```xml
<plugin>
    <artifactId>maven-compiler-plugin</artifactId>
    <version>3.5.1</version>
    <configuration>
        <source>11</source>
        <target>11</target>
        <!--<release>11</release>-->
    </configuration>
</plugin>
```

最终，运行maven编译通过。