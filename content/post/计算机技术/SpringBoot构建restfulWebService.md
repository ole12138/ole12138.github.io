---
title: "Spring构建restfulWebService"
date: 2021-01-06T11:18:01+08:00
draft: false
categories: 
- Spring
tags:
- Spring
- SpringBoot
- restfulWebService
---

# Spring构建restfulWebService

这是官网提供的一个例子，这里练习。

实际上这里用的是Springboot来创建的。

## What You Will Build

build a service that will accept HTTP GET requests at `http://localhost:8080/greeting`.

It will respond with a JSON representation of a greeting, as the following listing shows:

```json
{"id":1,"content":"Hello, World!"}
```

You can customize the greeting with an optional `name` parameter in the query string, as the following listing shows:

```json
http://localhost:8080/greeting?name=User
```

The `name` parameter value overrides the default value of `World` and is reflected in the response, as the following listing shows:

```json
{"id":1,"content":"Hello, User!"}
```

## 练习

### 初始化一个springboot项目

使用[springboot initializr](#https://start.spring.io/)来初始化一个springboot的web项目。

This example needs only the Spring Web dependency.

注意到项目的入口类为：

```java
package com.example.restfulWebService2;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class RestfulWebService2Application {

	public static void main(String[] args) {
		SpringApplication.run(RestfulWebService2Application.class, args);
	}

}
```

说明：

`@SpringBootApplication` is a convenience annotation that adds all of the following:

- `@Configuration`: Tags the class as a source of bean definitions for the application context.
- `@EnableAutoConfiguration`: Tells Spring Boot to start adding beans based on classpath settings, other beans, and various property settings. For example, if `spring-webmvc` is on the classpath, this annotation flags the application as a web application and activates key behaviors, such as setting up a `DispatcherServlet`.
- `@ComponentScan`: Tells Spring to look for other components, configurations, and services in the `com/example` package, letting it find the controllers.

The `main()` method uses Spring Boot’s `SpringApplication.run()` method to launch an application. Did you notice that there was not a single line of XML? There is no `web.xml` file, either. This web application is 100% pure Java and you did not have to deal with configuring any plumbing or infrastructure.

SpringBoot设置允许断点和调试：

在ide (idea)资源目录resouces下有application.properties文件，可以添加一行: 

```properties
debug=true
```

### Create a Resource Representation Class

创建一个实体类

```java
package com.example.restfulWebService2;

public class Greeting {
    public final long id;
    public final String content;

    public Greeting(long id, String content) {
        this.id = id;
        this.content = content;
    }

    public long getId() {
        return id;
    }

    public String getContent() {
        return content;
    }
}
```

### Create a Resource Controller

创建对应的controller来响应请求"/greeting"

```java
package com.example.restfulWebService2;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.ResponseBody;

import java.util.concurrent.atomic.AtomicLong;

@Controller
@ResponseBody
public class GreetingController {
    public static AtomicLong id = new AtomicLong();
    public static String template = "Hello, %s";

    @GetMapping(value = "/greeting")
    public Greeting greeting(@RequestParam(value = "name", defaultValue = "world") String name) {
        return new Greeting(id.incrementAndGet(), String.format(template, name));
    }
}
```

说明：

In Spring’s approach to building RESTful web services, HTTP requests are handled by a controller. These components are identified by the [`@RestController`](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/web/bind/annotation/RestController.html) annotation, and the `GreetingController` shown in the listing (from `src/main/java/com/example/restservice/GreetingController.java`) handles `GET` requests for `/greeting` by returning a new instance of the `Greeting` class).

This application uses the [Jackson JSON](https://github.com/FasterXML/jackson) library to automatically marshal instances of type `Greeting` into JSON. Jackson is included by default by the web starter.

The `@GetMapping` annotation ensures that HTTP GET requests to `/greeting` are mapped to the `greeting()` method.

 There are companion annotations for other HTTP verbs (e.g. `@PostMapping` for POST). There is also a `@RequestMapping` annotation that they all derive from, and can serve as a synonym (e.g. `@RequestMapping(method=GET)`).

`@RequestParam` binds the value of the query string parameter `name` into the `name` parameter of the `greeting()` method. If the `name` parameter is absent in the request, the `defaultValue` of `World` is used.

This code uses Spring [`@RestController`](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/web/bind/annotation/RestController.html) annotation, which marks the class as a controller where every method returns a domain object instead of a view. It is shorthand for including both `@Controller` and `@ResponseBody`.

The `Greeting` object must be converted to JSON. Thanks to Spring’s HTTP message converter support, you need not do this conversion manually. Because [Jackson 2](https://github.com/FasterXML/jackson) is on the classpath, Spring’s [`MappingJackson2HttpMessageConverter`](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/http/converter/json/MappingJackson2HttpMessageConverter.html) is automatically chosen to convert the `Greeting` instance to JSON.

### 测试

只有一个简单Java类和Controller，直接浏览器请求对应链接：
`http://localhost:8080/greeting?name=xxx`就可以了。