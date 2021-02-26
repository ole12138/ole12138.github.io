---
title: "Spring test 的使用"
date: 2021-02-10
draft: true
categories: 
- Java
series:
- 随笔
tags:
- Spring test
- TODO
---

# Spring test 的使用

官方文档位置：https://docs.spring.io/spring-framework/docs/current/reference/html/testing.html#integration-testing

#### 支持的注解

Spring的测试注释包括以下内容：

- [`@BootstrapWith`](https://docs.spring.io/spring-framework/docs/current/reference/html/testing.html#spring-testing-annotation-bootstrapwith)
- [`@ContextConfiguration`](https://docs.spring.io/spring-framework/docs/current/reference/html/testing.html#spring-testing-annotation-contextconfiguration)
- [`@WebAppConfiguration`](https://docs.spring.io/spring-framework/docs/current/reference/html/testing.html#spring-testing-annotation-webappconfiguration)
- [`@ContextHierarchy`](https://docs.spring.io/spring-framework/docs/current/reference/html/testing.html#spring-testing-annotation-contexthierarchy)
- [`@ActiveProfiles`](https://docs.spring.io/spring-framework/docs/current/reference/html/testing.html#spring-testing-annotation-activeprofiles)
- [`@TestPropertySource`](https://docs.spring.io/spring-framework/docs/current/reference/html/testing.html#spring-testing-annotation-testpropertysource)
- [`@DynamicPropertySource`](https://docs.spring.io/spring-framework/docs/current/reference/html/testing.html#spring-testing-annotation-dynamicpropertysource)
- [`@DirtiesContext`](https://docs.spring.io/spring-framework/docs/current/reference/html/testing.html#spring-testing-annotation-dirtiescontext)
- [`@TestExecutionListeners`](https://docs.spring.io/spring-framework/docs/current/reference/html/testing.html#spring-testing-annotation-testexecutionlisteners)
- [`@RecordApplicationEvents`](https://docs.spring.io/spring-framework/docs/current/reference/html/testing.html#spring-testing-annotation-recordapplicationevents)
- [`@Commit`](https://docs.spring.io/spring-framework/docs/current/reference/html/testing.html#spring-testing-annotation-commit)
- [`@Rollback`](https://docs.spring.io/spring-framework/docs/current/reference/html/testing.html#spring-testing-annotation-rollback)
- [`@BeforeTransaction`](https://docs.spring.io/spring-framework/docs/current/reference/html/testing.html#spring-testing-annotation-beforetransaction)
- [`@AfterTransaction`](https://docs.spring.io/spring-framework/docs/current/reference/html/testing.html#spring-testing-annotation-aftertransaction)
- [`@Sql`](https://docs.spring.io/spring-framework/docs/current/reference/html/testing.html#spring-testing-annotation-sql)
- [`@SqlConfig`](https://docs.spring.io/spring-framework/docs/current/reference/html/testing.html#spring-testing-annotation-sqlconfig)
- [`@SqlMergeMode`](https://docs.spring.io/spring-framework/docs/current/reference/html/testing.html#spring-testing-annotation-sqlmergemode)
- [`@SqlGroup`](https://docs.spring.io/spring-framework/docs/current/reference/html/testing.html#spring-testing-annotation-sqlgroup)

## 简单使用

#### 首先要配置maven

pom.xml文件中引入spring-test.jar的依赖：

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-test</artifactId>
    <version>5.3.3</version>
    <scope>test</scope>
</dependency>
```

如果使用gradle，配置类似。

#### 开启带有Spring IOC容器的测试集

开启一个带有spring IOC容器的测试集：

如果使用xml形式的spring容器配置：

```java
@ContextConfiguration("/test-config.xml") 
class XmlApplicationContextTests {
    // class body...
}
```

如果使用JavaConfig形式的spring容器配置：

```java
@ContextConfiguration(classes = TestConfig.class) 
class ConfigClassApplicationContextTests {
    // class body...
}
```

#### web环境的测试要启动web上下文

需要在类上附加@WebAppConfiguration

支持`classpath:`或`file:`资源前缀。

```java
@ContextConfiguration
@WebAppConfiguration("classpath:test-web-resources") 
class WebAppTests {
    // class body...
}
```

#### 指定生效的Profile

需要在类上附加@ActiveProfiles

```java
@ContextConfiguration
@ActiveProfiles("dev") 
class DeveloperTests {
    // class body...
}
```

#### 使用属性文件

需要在类上附加`@TestPropertySource`

```java
@ContextConfiguration
@TestPropertySource("/test.properties") 
class MyIntegrationTests {
    // class body...
}
```

PropertySource属于Environment抽象，有些翻译里称作内联属性，优先级比JavaVM变量和系统变量高。

属性文件里定义的key-value就可以通过placeholder形式：`${xxx.xxx}`使用了

#### 使用动态属性

在方法上附加`@DynamicPropertySource`

可用于注册 要添加到的集合中以用于集成测试的*动态*属性。当不预先知道属性的值时，例如，如果属性是由外部资源管理的，会很有用。

没作尝试，有需要再说。

#### 刷新基础Spring容器

类上或方法上加`@DirtiesContext`

还分为方法使用前，方法使用后，上下文的变脏刷新。

没作尝试，有需要再说。一般来说，如果测试加了事物回滚，是不需要刷新的。

#### 记录事件

在类上附加`@RecordApplicationEvents`。指示 *Spring TestContext Framework*记录`ApplicationContext`在单个测试的执行期间发布的所有应用程序事件 。

可以`ApplicationEvents`在测试中通过API访问记录的事件。

从Spring Framework 5.3.3开始，TestContext框架提供了对记录在中发布的 [应用程序事件的](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#context-functionality-events)支持， `ApplicationContext`以便可以针对测试中的那些事件执行断言。通过`ApplicationEvents`API可以执行在单个测试执行过程中发布的所有事件，该事件使您可以将事件处理为 `java.util.Stream`。

要`ApplicationEvents`在测试中使用，请执行以下操作。

- 确保您的测试类已使用注释或进行了元注释 [`@RecordApplicationEvents`](https://docs.spring.io/spring-framework/docs/current/reference/html/testing.html#spring-testing-annotation-recordapplicationevents)。
- 确保`ApplicationEventsTestExecutionListener`已注册。但是请注意，它`ApplicationEventsTestExecutionListener`是默认注册的，并且只有在您具有`@TestExecutionListeners`不包含默认侦听器的自定义配置的情况下，才需要手动注册 。
- 注释类型的字段`ApplicationEvents`与`@Autowired`和使用该实例 `ApplicationEvents`在您的测试和生命周期方法

```java
@SpringJUnitConfig(/* ... */)
@RecordApplicationEvents 
class OrderServiceTests {

    @Autowired
    OrderService orderService;

    @Autowired
    ApplicationEvents events; 

    @Test
    void submitOrder() {
        // Invoke method in OrderService that publishes an event
        orderService.submitOrder(new Order(/* ... */));
        // Verify that an OrderSubmitted event was published
        int numEvents = events.stream(OrderSubmitted.class).count(); 
        assertThat(numEvents).isEqualTo(1);
    }
}
```



有关示例和更多详细信息，请参见[应用程序事件](https://docs.spring.io/spring-framework/docs/current/reference/html/testing.html#testcontext-application-events)和 [`@RecordApplicationEvents` javadoc](https://docs.spring.io/spring-framework/docs/5.3.3/javadoc-api/org/springframework/test/context/event/RecordApplicationEvents.html)。

#### 提交事物

`@Commit`表示应该在测试方法完成后提交用于事务测试方法的事务。可直接替换`@Rollback(false)`以更明确地传达代码的意图。与相似`@Rollback`，`@Commit`也可以声明为类级别或方法级别的注释。

```java
@Commit 
@Test
void testProcessWithoutRollback() {
    // ...
}
```

#### 事物回滚

`@Rollback`指示是否应在测试方法完成后回退事务测试方法的事务。如果为`true`，则事务将回滚。否则，将提交事务（另请参见 [`@Commit`](https://docs.spring.io/spring-framework/docs/current/reference/html/testing.html#spring-testing-annotation-commit)）。即使`@Rollback`未明确声明，也有`@Rollback(true)`的默认行为。

当声明为类级注释时，`@Rollback`为测试类层次结构内的所有测试方法定义默认回滚语义。当声明为方法级注释时，`@Rollback`为特定的测试方法定义回滚语义，可能覆盖类级`@Rollback`或`@Commit`语义。

```java
@Rollback(true) 
@Test
void testProcessWithoutRollback() {
    // ...
}
```

#### 其他测试注解

`@AfterTransaction`表示`void`对于已配置为使用Spring`@Transactional`注释在事务内运行的测试方法，在事务结束后应运行带注释的方法。

`@Sql`用于注释测试类或测试方法，以配置在集成测试期间针对给定数据库运行的SQL脚本。

`@SqlConfig`定义用于确定如何解析和运行使用`@Sql`注释配置的SQL脚本的元数据。

`@SqlMergeMode`用于注释测试类或测试方法，以配置是否将方法级`@Sql`声明与类级`@Sql`声明合并。

`@SqlGroup`是一个聚合多个`@Sql`注释的容器注释。

#### 标准注解支持

- `@Autowired`
- `@Qualifier`
- `@Value`
- `@Resource` （javax.annotation）如果存在JSR-250
- `@ManagedBean` （javax.annotation）如果存在JSR-250
- `@Inject` （javax.inject）如果存在JSR-330
- `@Named` （javax.inject）如果存在JSR-330
- `@PersistenceContext` （javax.persistence）如果存在JPA
- `@PersistenceUnit` （javax.persistence）如果存在JPA
- `@Required`
- `@Transactional`（org.springframework.transaction.annotation） *具有[有限的属性支持](https://docs.spring.io/spring-framework/docs/current/reference/html/testing.html#testcontext-tx-attribute-support)*

## Spring test和JUnit jupiter (JUnit5)的配合使用

#### 首先要配置maven

引入spring-test.jar和junit-jupter.jar的依赖

```xml
<dependency>
   <groupId>org.springframework</groupId>
    <artifactId>spring-test</artifactId>
    <version>5.3.3</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter</artifactId>
    <version>RELEASE</version>
    <scope>test</scope>
</dependency>
```

如果使用gradle，配置类似。

#### 开启带有Spring IOC容器的测试集

在类上附加`@SpringJUnitConfig`

`@SpringJUnitConfig`是由JUnit Jupiter的`@ExtendWith(SpringExtension.class)`注解和Spring test的`@ContextConfiguration`注解组合而成的组合注释 。

它是`@ContextConfiguration`的替代。

关于配置选项，`@ContextConfiguration`和之间的唯一区别是`@SpringJUnitConfig`默认的`value`属性用于声明JavaConfig，而`@ContextConfiguration`默认的`value`属性用于声明xml配置文件。

Spring容器使用JavaConfig配置时：

```java
@SpringJUnitConfig(TestConfig.class) 
class ConfigurationClassJUnitJupiterSpringTests {
    // class body...
}
```

Spring容器使用xml配置时：

```java
@SpringJUnitConfig(locations = "/test-config.xml") 
class XmlJUnitJupiterSpringTests {
    // class body...
}
```

#### web环境要启动web上下文

类上附加`@SpringJUnitWebConfig`

`@SpringJUnitWebConfig`是由JUnit Jupiter的`@ExtendWith(SpringExtension.class)`注解和Spring Test的`@ContextConfiguration`以及 `@WebAppConfiguration`组合而成的组合注释 。

可以用它替换掉Spring test的`@ContextConfiguration`和`@WebAppConfiguration`。

只是`@SpringJUnitWebConfig`的配置选项`value`属性默认用来声明JavaConfig配置文件。

如果使用JavaConfig配置SpringWeb上下文：

```java
@SpringJUnitWebConfig(TestConfig.class) 
class ConfigurationClassJUnitJupiterSpringWebTests {
    // class body...
}
```

如果使用xml来配置SpringWeb上下文：

```xml
@SpringJUnitWebConfig(locations = "/test-config.xml") 
class XmlJUnitJupiterSpringWebTests {
    // class body...
}
```

#### 其他

`@TestConstructor`是一个类级别的注解，用来配置如何将测试的`ApplicationContext`组件参数装配到测试类构造方法的参数中。如果`@TestConstructor`不存在，那么会有一个默认的装配模式被使用。

`@NestedTestConfiguration`是一个类级别的注解，被用来设置Spring测试配置注解如何在内部测试类中运行。默认的封闭配置继承模式是`INHERIT`。从JUnit Jupter启动时，用前面介绍的Spring注解时，可能会需要这个注解。

`@EnabledIf`表示它修饰的JUnit Jupiter类或者测试方法是否启用，由提供的`expression`结果决定。具体来说，如果一个表达式的计算结果是`Boolean.TRUE`或者一个`String`equal为`true`(忽略大小写)，这个测试就是启用的。当应用于类级别时，所有在该类中的测试方法都会默认启用。
支持：SpEL表达式模板形式`#{}`、Spring的placeholder形式`${}`、文本`true`或`false`
例如：测试方法上的`@EnableIf("true")`表示永远都会启动测试

`@DisabledIf`略

还可以用这些介绍过的注解来自定义组合注解，略。

## Spring test框架结构的介绍

TODO

官方文档位置：https://docs.spring.io/spring-framework/docs/current/reference/html/testing.html#testcontext-framework

## WebTestClient测试服务器应用程序

TODO

`WebTestClient`是Spring test提供的，用于**测试服务器应用程序**的HTTP客户端。

官方文档位置：https://docs.spring.io/spring-framework/docs/current/reference/html/testing.html#webtestclient

## MockMvc测试mvc应用程序

TOOD

`MockMvc`是Spring test提供的，用于**测试Spring MVC应用程序**的测试工具。

它通过模拟请求和响应对象而不是正在运行的服务器执行完整的Spring MVC请求处理。

官方文档位置：https://docs.spring.io/spring-framework/docs/current/reference/html/testing.html#spring-mvc-test-framework

## MockRestServiceServer测试客户应用程序

TODO

也可以使用spring test框架**测试客户端应用**，它内部使用的是`RestTemplate`。这个逻辑是申明期待的请求和提供"stub"response，所以可以在不运行服务的情况下检测代码。

官方文档位置：https://docs.spring.io/spring-framework/docs/current/reference/html/testing.html#spring-mvc-test-client



官方文档可以右键谷歌翻译，翻译质量还可以。
有一篇翻译不错的人工翻译：http://www.mingaccount.com/2017/09/spring%E6%B5%8B%E8%AF%95/