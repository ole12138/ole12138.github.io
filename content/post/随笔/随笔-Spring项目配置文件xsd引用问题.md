---
title: "随笔-Spring项目配置文件xsd引用问题"
date: 2021-01-21T17:53:27+08:00
draft: false
categories: 
- Java
- Java随笔
tags:
- Spring配置文件的xsd引用问题
---

# 随笔-Spring项目配置文件xsd引用问题

新建一个Spring项目（从XML配置文件），大概这样：

```java
public class Main {
    public static void main(String[] args) {
        ApplicationContext applicationContext = new ClassPathXmlApplicationContext("applicationContext.xml");
        //随便一个bean
        Message message = (Message) applicationContext.getBean("message");
        System.out.println(message);
    }
}
```

然鹅却迟迟不能启动，好一会儿控制台报错：

```
[main] WARN  org.springframework.beans.factory.xml.XmlBeanDefinitionReader - Ignored XML validation warning
org.xml.sax.SAXParseException: schema_reference.4: Failed to read schema document '/spring-context-4.0.xsd', because 1) could not find the document; 2) the document could not be read; 3) the root element of the document is not <xsd:schema>.
...
```

Spring配置文件是xml格式的，其中的标签需要对应的说明文档。这也就是Spring配置文件的格式说明文档没有加载上呗。

开始以为是网络问题，但是查了以下资料，发现就算网络不通，Spring也是优先从本地加载说明文档。

spring-context.jar下的META-INF下也是有spring.schema文件的，并在其中将远程链接重定向到了本地jar包里的xsd说明文档：

```
http\://www.springframework.org/schema/context/spring-context.xsd=org/springframework/context/config/spring-context.xsd
```

仔细核对才发现是自己将xsd的链接写错了，并没能重定向到本地，也没有在远程找到对应的文件。