---
title: "创建Spring项目（XML方式）"
date: 2021-01-05T12:00:39+08:00
draft: true
categories: 
- Spring
tags:
- Spring
---

# 认识Spring

Spring一般指Spring Framework。

The Spring Framework is divided into modules. Applications can choose which modules they need.

 At the heart are the modules of the **core container**, including **a configuration model** and **a dependency injection mechanism**. 

即Spring是个java框架（或者说容器），包括了用于接收配置的模型，以及依赖注入机制。

利用Spring，可以把众多的Java组件（第三方的、自己实现的、Spring提供的等等）的实例（默认是单例）组合起来，方便地引用与调用，而不需要太多的连接代码。

Beyond that, the Spring Framework provides foundational support for different application architectures, including messaging, transactional data and persistence, and web. It also includes the Servlet-based Spring MVC web framework and, in parallel, the Spring WebFlux reactive web framework.

## Spring与JavaEE

Spring并不拥抱JavaEE标准，但是选择性的实现了其中几个独立的标准。

## Spring是一个管理组件的容器

The Spring Framework implements of the Inversion of Control (IoC) principle. IoC is also known as dependency injection (DI). It is a process whereby objects define their dependencies (that is, the other objects they work with) only through constructor arguments, arguments to a factory method, or properties that are set on the object instance after it is constructed or returned from a factory method. The container then injects those dependencies when it creates the bean. This process is fundamentally the inverse (hence the name, Inversion of Control) of the bean itself controlling the instantiation or location of its dependencies by using direct construction of classes or a mechanism such as the Service Locator pattern.

IOC控制反转就是说，各个类创建实例（对象）的过程，交给Spring容器来完成。

DI依赖注入就是说，类的实例（对象）中的依赖（用到的其他实例的引用），由Spring容器来传入。



![容器魔术](https://picgo12138.oss-cn-hangzhou.aliyuncs.com/md/container-magic.png)

上图说明了Spring容器的作用：读取配置文件(configuration metadata), 实例化POJO等，加入Spring容器，并为其自动注入依赖。

# 创建Spring项目（使用XML配置）

Spring文档：https://docs.spring.io/spring-framework/docs/current/reference/html/

Spring文档中文翻译：https://www.docs4dev.com/docs/zh/spring-framework/4.3.21.RELEASE/reference

想使用Spring，首先关注它的核心部件的文档：
[Core](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#spring-core)：IoC Container, Events, Resources, i18n, Validation, Data Binding, Type Conversion, SpEL, AOP.

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

一个典型的配置文件

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

## 启动Spring

例如，在com.jingmin包下有个MyBean类，有个函数doSomething()，表示可以完成一定的业务逻辑。

```java
package com.jingmin;
public class MyBean {
    public void doSomething() {
        System.out.println("do something by MyBean");
    }
}
```

在项目的classpath下，创建一个xml配置文件applicationContext.xml，在此xml头部加入格式说明的链接，使其可被Spring识别：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd">
    
    <bean id="myBean" class="com.jingmin.MyBean"/>
</beans>
```

这里的配置文件声明了一个bean，它的类型是com.jingmin.MyBean。

项目下创建一个入口类，启动Spring：

```java
package com.jingmin;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
public class Main {
	public static void main(String[] args) {
        //传入配置文件，启动Spring
        ApplicationContext applicationContext = new ClassPathXmlApplicationContext("applicationContext.xml");
        //Spring会根据配置文件实例化一个MyBean的实例，这里把这个实例取出来
        MyBean myBean = applicationContext.getBean("myBean", MyBean.class);
        //完成一定的业务逻辑
        myBean.doSomething();
    }
}
```

Indeed, your application code should have no calls to the getBean() method at all and thus have no dependency on Spring APIs at all. 

## 依赖注入

上面的例子中，Spring根据配置文件，声明式实例化MyBean的过程中，MyBean并没有依赖：

```xml
<bean id="myBean" class="com.jingmin.MyBean"/>
```



`ApplicationContext`支持其 Management 的 bean 的基于构造函数和基于 setter 的 DI。在已经通过构造函数方法注入了某些依赖项之后，它还支持基于 setter 的 DI。

您可以以`BeanDefinition`的形式配置依赖项，并与`PropertyEditor`实例结合使用以将属性从一种格式转换为另一种格式。但是，大多数 Spring 用户并不直接(即以编程方式)使用这些类，
而是使用 XML `bean`定义，带 Comments 的组件(即以`@Component`，`@Controller`等标记的类)或基于 Java 的`@Configuration`类中的`@Bean`方法。然后将这些源在内部转换为`BeanDefinition`的实例，并用于加载整个 Spring IoC 容器实例。

可以混合使用基于构造函数的 DI 和基于 setter 的 DI，因此，将构造函数用于*强制依赖项*和将 setter 方法或配置方法用于*可选依赖项*是一个很好的经验法则。请注意，在 setter 方法上使用[@Required](https://www.docs4dev.com/docs/zh/spring-framework/4.3.21.RELEASE/reference/beans.html#beans-required-annotation)注解可用于使属性成为必需的依赖项。

Setter 注入主要应仅用于可以在类中分配合理的默认值的可选依赖项。否则，必须在代码使用依赖项的任何地方执行非空检查。 setter 注入的一个好处是，setter 方法使该类的对象在以后可以重新配置或重新注入。

如果主要使用构造函数注入，则可能会创建无法解决的循环依赖方案。

例如：A 类通过构造函数注入需要 B 类的实例，而 B 类通过构造函数注入需要 A 类的实例。如果为将类 A 和 B 相互注入而配置了 bean，则 Spring IoC 容器会在运行时检测到此循环引用，并抛出`BeanCurrentlyInCreationException`。

一种可能的解决方案是编辑某些类的源代码，这些类的源代码由设置者而不是构造函数来配置。或者，避免构造函数注入，而仅使用 setter 注入。换句话说，尽管不建议这样做，但是您可以使用 setter 注入配置循环依赖关系。

与“典型”情况(没有循环依赖)不同，bean A 和 bean B 之间的循环依赖迫使其中一个 bean 在完全完全初始化之前被注入另一个 bean(经典的 chicken/egg 场景)。

实际上，后面的Spring项目中，应当配置注解自动扫描，然后在相应的包下直接使用注解直接注入Bean。（还需要引入spring-context的标签格式说明文档.)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd">
    <context:component-scan base-package="com.jingmin.*"/>
</beans>
```

## 