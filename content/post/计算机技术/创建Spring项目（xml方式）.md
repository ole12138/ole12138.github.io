---
title: "创建Spring项目（XML方式）"
date: 2021-01-05T12:00:39+08:00
draft: true
categories: 
- Spring
tags:
- Spring
---

# Spring Framework的认识

The Spring Framework is divided into modules. Applications can choose which modules they need.

 At the heart are the modules of the **core container**, including **a configuration model** and **a dependency injection mechanism**. 

即Spring是个java框架（或者说容器），包括了用于接收配置的模型，以及依赖注入机制。

利用Spring，可以把众多的Java组件（第三方的、自己实现的、Spring提供的等等）的实例（默认是单例）组合起来，方便地引用与调用，而不需要太多的连接代码。

Beyond that, the Spring Framework provides foundational support for different application architectures, including messaging, transactional data and persistence, and web. It also includes the Servlet-based Spring MVC web framework and, in parallel, the Spring WebFlux reactive web framework.

## Spring与JavaEE

Spring并不拥抱JavaEE标准，但是选择性的实现了其中几个独立的标准。

# Maven创建Spring项目（并配置XML）

Spring文档：https://docs.spring.io/spring-framework/docs/current/reference/html/

想使用Spring，首先关注它的核心部件的文档：
[Core](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#spring-core)：IoC Container, Events, Resources, i18n, Validation, Data Binding, Type Conversion, SpEL, AOP.

## Spring是一个管理组件的容器

The Spring Framework implements of the Inversion of Control (IoC) principle. IoC is also known as dependency injection (DI). It is a process whereby **objects define their dependencies** (that is, the other objects they work with) **only through constructor arguments, arguments to a factory method, or properties that are set on the object instance after it is constructed or returned from a factory method**. **The container then injects those dependencies when it creates the bean**. This process is fundamentally the inverse (hence the name, Inversion of Control) of the bean itself controlling the instantiation or location of its dependencies by using direct construction of classes or a mechanism such as the Service Locator pattern.

The following diagram shows a high-level view of how Spring works. Your application classes are combined with configuration metadata so that, after the `ApplicationContext` is created and initialized, you have a fully configured and executable system or application.

![容器魔术](https://picgo12138.oss-cn-hangzhou.aliyuncs.com/md/container-magic.png)

## 创建Maven项目并引入Spring依赖

参考spring project 代码主页的[说明](https://github.com/spring-projects/spring-framework/wiki/Spring-Framework-Artifacts)，在Maven的pom文件中引入Spring：

```xml
<!-- https://mvnrepository.com/artifact/org.springframework/spring-context -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>5.2.12.RELEASE</version>
        </dependency>
```

## 在xml中配置Spring的Bean

在Spring中，构成应用程序主干并由Spring IoC容器管理的对象称为bean。Bean是由Spring IoC容器实例化，组装和管理的对象。否则，bean仅仅是应用程序中许多对象之一。Bean及其之间的依赖关系反映在容器使用的配置文件中。

早期的Spring都使用xml作为配置文件，一般放到项目类路径的根目录下，方便使用。

一个典型的

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="..." class="...">  
        <!-- collaborators and configuration for this bean go here -->
    </bean>

    <bean id="..." class="...">
        <!-- collaborators and configuration for this bean go here -->
    </bean>

    <!-- more bean definitions go here -->

</beans>
```

The `id` attribute is a string that identifies the individual bean definition.

The `class` attribute defines the type of the bean and uses the fully qualified classname

当一个配置文件中bean较多时，可以分为多个配置文件：

```xml
<beans>
    <import resource="services.xml"/>
    <import resource="resources/messageSource.xml"/>
    <import resource="/resources/themeSource.xml"/>

    <bean id="bean1" class="..."/>
    <bean id="bean2" class="..."/>
</beans>
```



注意： XML-based metadata is not the only allowed form of configuration metadata. The Spring IoC container itself is totally decoupled from the format in which this configuration metadata is actually written. These days, many developers choose [Java-based configuration](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-java) for their Spring applications。

## 在main()函数中启动Spring对Bean的管理

The `org.springframework.beans` and `org.springframework.context` packages are the basis for Spring Framework’s IoC container. The [`BeanFactory`](https://docs.spring.io/spring-framework/docs/5.3.2/javadoc-api/org/springframework/beans/factory/BeanFactory.html) interface provides an advanced configuration mechanism capable of managing any type of object. [`ApplicationContext`](https://docs.spring.io/spring-framework/docs/5.3.2/javadoc-api/org/springframework/context/ApplicationContext.html) is a sub-interface of `BeanFactory`.

Several implementations of the `ApplicationContext` interface are supplied with Spring. In stand-alone applications, it is common to create an instance of [`ClassPathXmlApplicationContext`](https://docs.spring.io/spring-framework/docs/5.3.2/javadoc-api/org/springframework/context/support/ClassPathXmlApplicationContext.html) or [`FileSystemXmlApplicationContext`](https://docs.spring.io/spring-framework/docs/5.3.2/javadoc-api/org/springframework/context/support/FileSystemXmlApplicationContext.html). 

> 上面也提到过，除了可以使用xml配置文件，也可以使用Java注解来配置：While XML has been the traditional format for defining configuration metadata, you can instruct the container to use Java annotations or code as the metadata format by providing a small amount of XML configuration to declaratively enable support for these additional metadata formats.
>

The `ApplicationContext` is the interface for an advanced factory capable of maintaining a registry of different beans and their dependencies. By using the method `T getBean(String name, Class<T> requiredType)`, you can retrieve instances of your beans.

The `ApplicationContext` lets you read bean definitions and access them。例如，在程序的入口main()中加入如下代码，则将开启Spring，并从类路径下寻找对应的xml配置文件，依据配置，开始对加入Spring的各种Bean进行管理。

```java
// create and configure beans
ApplicationContext context = new ClassPathXmlApplicationContext("services.xml", "daos.xml");

// retrieve configured instance
PetStoreService service = context.getBean("petStore", PetStoreService.class);

// use configured instance
List<String> userList = service.getUsernameList();
```

You can then use `getBean` to retrieve instances of your beans. The `ApplicationContext` interface has a few other methods for retrieving beans, but, ideally, your application code should never use them. Indeed, your application code **should have no calls to the `getBean()` method at all and thus have no dependency on Spring APIs at al**l. For example, Spring’s integration with web frameworks provides dependency injection for various web framework components such as controllers and JSF-managed beans, letting you declare a dependency on a specific bean through metadata (such as an autowiring annotation).

实际上，后面的Spring项目中，应当配置注解自动扫描，然后在相应的包下直接使用注解直接注入Bean。