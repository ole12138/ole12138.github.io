---
title: "随笔-Spring及SpringMVC配置文件中annotation-config和annotation-driven和component-scan的区别（转载）"
date: 2021-01-28T20:00:58+08:00
draft: false
categories: 
- Java
- Java随笔
tag:
- annotation-config
- annotation-driven
- component-scan
---

# annotation-config和annotation-driven和component-scan的区别

本章非原创，转载来源：https://blog.csdn.net/catoop/article/details/50068573

本文开门见山，直接分别进行解释：
**一、`<context:annotation-config/>`**
隐式地向Spring容器中注册AutowiredAnnotationBeanPostProcessor、CommonAnnotationBeanPostProcessor、PersistenceAnnotationBeanPostProcessor 及 equiredAnnotationBeanPostProcessor 这 4 个 BeanPostProcessor
对这个结果类做个解释：
1、如果你想使用@Autowired注解，那么就必须事先在 Spring 容器中声明 AutowiredAnnotationBeanPostProcessor Bean。
2、如果想使用@ Resource 、@ PostConstruct、@ PreDestroy等注解就必须声明CommonAnnotationBeanPostProcessor。
3、如果想使用@PersistenceContext注解，就必须声明PersistenceAnnotationBeanPostProcessor的Bean。
4、如果想使用 @Required的注解，就必须声明RequiredAnnotationBeanPostProcessor的Bean。
分别对应的传统声明方式为：

```
<bean class="org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor"/> 
<bean class="org.springframework.context.annotation.CommonAnnotationBeanPostProcessor"/> 
<bean class="org.springframework.orm.jpa.support.PersistenceAnnotationBeanPostProcessor"/> 
<bean class="org.springframework.beans.factory.annotation.RequiredAnnotationBeanPostProcessor"/> 1234
```

一般来说，这些注解我们还是比较常用，尤其是Antowired的注解，在自动注入的时候更是经常使用，所以如果总是需要按照传统的方式一条一条配置显得有些繁琐和没有必要，于是spring给我们提供 `<context:annotation-config>` 的简化配置方式，自动帮你完成声明。

`<context:annotation-config/>` 将隐式地向 Spring 容器注册 AutowiredAnnotationBeanPostProcessor、CommonAnnotationBeanPostProcessor、PersistenceAnnotationBeanPostProcessor 以及 equiredAnnotationBeanPostProcessor 这 4 个 BeanPostProcessor。

**二、 `<context:component-scan/>`**
`<context:component-scan/>` 配置项不但启用了对类包进行扫描以实施注释驱动 Bean 定义的功能，同时还启用了注释驱动自动注入的功能（即还隐式地在内部注册了 AutowiredAnnotationBeanPostProcessor 和 CommonAnnotationBeanPostProcessor），因此当使用 `<context:component-scan base-package="xxx.xxx.xxx"/>` 后，就可以将 `<context:annotation-config/>` 移除了。

默认情况下通过 @Component 定义的 Bean 都是 singleton 的，如果需要使用其它作用范围的 Bean，可以通过 @Scope 注释来达到目标，如：@Scope(“prototype”)，这样，当从 Spring 容器中获取 boss Bean 时，每次返回的都是新的实例了。

**三、`<mvc:annotation-driven/>`**
相当于注册了DefaultAnnotationHandlerMapping和AnnotationMethodHandlerAdapter两个bean，配置一些messageconverter。即解决了@Controller注解的使用前提配置。

官方文档中也进行了说明，如下：
`<mvc:annotation-driven/>` is a tag added in Spring 3.0 which does the following:

1. Configures the Spring 3 Type ConversionService (alternative to PropertyEditors)

2. Adds support for formatting Number fields with @NumberFormat

3. Adds support for formatting Date, Calendar, and Joda Time fields with @DateTimeFormat, if Joda Time is on the classpath

4. Adds support for validating @Controller inputs with @Valid, if a JSR-303 Provider is on the classpath

5. Adds support for support for reading and writing XML, if JAXB is on the classpath (HTTP message conversion with @RequestBody/@ResponseBody)

6. Adds support for reading and writing JSON, if Jackson is o n the classpath (along the same lines as #5)

   `<context:annotation-config/>`
   Looks for annotations on beans in the same application context it is defined and declares support for all the general annotations like @Autowired, @Resource, @Required, @PostConstruct etc etc.
   `<context:annotation-config>` does NOT search for @Component, @Controller, etc.
   `<context:component-scan>` DOES search for those @Component annotations, as well as the annotations that `<context:annotation-config/>` does.
   there are other “annotation-driven” tags available to provide additional functionality in other Spring modules. For example, `<transaction:annotation-driven />` enables the use of the @Transaction annotation, `<task:annotation-driven />` is required for @Scheduled et al

# 源码中相关代码

在Spring的启动refresh过程中，首先进行的是xml中bean标签的解析，`<context:annotation-config/>`和`<context:component-scan/>`作为自定义标签，也会在这个过程中被解析处理。

首先定位到AbstractApplicationContext的refresh函数。

```java
@Override
public void refresh() throws BeansException, IllegalStateException {
   ...
      // Prepare this context for refreshing.
      prepareRefresh();

      // Tell the subclass to refresh the internal bean factory.
      ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();
      
      ...
}
```

从obtainFreshBeanFactory()跟进去看，一直到跟进到委托类BeanDefinitionParserDelegate中处理自定义标签的位置：

```java
@Nullable
public BeanDefinition parseCustomElement(Element ele, @Nullable BeanDefinition containingBd) {
   String namespaceUri = getNamespaceURI(ele);
   if (namespaceUri == null) {
      return null;
   }
   NamespaceHandler handler = this.readerContext.getNamespaceHandlerResolver().resolve(namespaceUri);
   if (handler == null) {
      error("Unable to locate Spring NamespaceHandler for XML schema namespace [" + namespaceUri + "]", ele);
      return null;
   }
   return handler.parse(ele, new ParserContext(this.readerContext, this, containingBd));
}
```

查看相关代码，`this.readerContext.getNamespaceHandlerResolver()`默认获取的是 DefaultNamespaceHandlerResolver的一个实例。查看它的resolve方法

```java
@Override
@Nullable
public NamespaceHandler resolve(String namespaceUri) {
   //查看相关代码可以发现，通过查找spring.handlers文件，
   //在handlerMappings中注册了context命名空间的处理器ContextNamespaceHandler
   Map<String, Object> handlerMappings = getHandlerMappings();
   Object handlerOrClassName = handlerMappings.get(namespaceUri);
   if (handlerOrClassName == null) {
      return null;
   }
   else if (handlerOrClassName instanceof NamespaceHandler) {
      return (NamespaceHandler) handlerOrClassName;
   }
   else {
      String className = (String) handlerOrClassName;
      try {
         Class<?> handlerClass = ClassUtils.forName(className, this.classLoader);
         if (!NamespaceHandler.class.isAssignableFrom(handlerClass)) {
            throw new FatalBeanException("Class [" + className + "] for namespace [" + namespaceUri +
                  "] does not implement the [" + NamespaceHandler.class.getName() + "] interface");
         }
         //第一次使用handler要先实例化
         NamespaceHandler namespaceHandler = (NamespaceHandler) BeanUtils.instantiateClass(handlerClass);
         //调用初始化方法，如果是context命名空间的处理器ContextNamespaceHandler，也会调用相应的init方法
         namespaceHandler.init();
         handlerMappings.put(namespaceUri, namespaceHandler);
         return namespaceHandler;
      }
      catch (ClassNotFoundException ex) {
         throw new FatalBeanException("Could not find NamespaceHandler class [" + className +
               "] for namespace [" + namespaceUri + "]", ex);
      }
      catch (LinkageError err) {
         throw new FatalBeanException("Unresolvable class definition for NamespaceHandler class [" +
               className + "] for namespace [" + namespaceUri + "]", err);
      }
   }
}
```

查看ContextNamespaceHandler的init方法

```java
public class ContextNamespaceHandler extends NamespaceHandlerSupport {
   @Override
   public void init() {
      registerBeanDefinitionParser("property-placeholder", new PropertyPlaceholderBeanDefinitionParser());
      registerBeanDefinitionParser("property-override", new PropertyOverrideBeanDefinitionParser());
      registerBeanDefinitionParser("annotation-config", new AnnotationConfigBeanDefinitionParser());
      registerBeanDefinitionParser("component-scan", new ComponentScanBeanDefinitionParser());
      registerBeanDefinitionParser("load-time-weaver", new LoadTimeWeaverBeanDefinitionParser());
      registerBeanDefinitionParser("spring-configured", new SpringConfiguredBeanDefinitionParser());
      registerBeanDefinitionParser("mbean-export", new MBeanExportBeanDefinitionParser());
      registerBeanDefinitionParser("mbean-server", new MBeanServerBeanDefinitionParser());
   }
}
```

可以看到这里注册了处理context标签下各个属性的parser。`<context:annotation-config/>`的解析器是AnnotationConfigBeanDefinitionParser，`<context:component-scan/>`的解析器是ComponentScanBeanDefinitionParser。

返回到委托类BeanDefinitionParserDelegate中处理自定义标签的位置：

```java
@Nullable
public BeanDefinition parseCustomElement(Element ele, @Nullable BeanDefinition containingBd) {
   String namespaceUri = getNamespaceURI(ele);
   if (namespaceUri == null) {
      return null;
   }
   //对于context命名空间，返回的是ContextNamespaceHandler的实例
   NamespaceHandler handler = this.readerContext.getNamespaceHandlerResolver().resolve(namespaceUri);
   if (handler == null) {
      error("Unable to locate Spring NamespaceHandler for XML schema namespace [" + namespaceUri + "]", ele);
      return null;
   }
   return handler.parse(ele, new ParserContext(this.readerContext, this, containingBd));
}
```

可知，对于context命名空间，返回的handler是ContextNamespaceHandler的实例。

查看此handler的parse方法，发现在父类NamespaceHandlerSupport中定义

```java
@Override
@Nullable
public BeanDefinition parse(Element element, ParserContext parserContext) {
   //根据属性名不同，返回不同的parser
   BeanDefinitionParser parser = findParserForElement(element, parserContext);
   //调用对应parser的parse方法
   return (parser != null ? parser.parse(element, parserContext) : null);
}

/**
 * Locates the {@link BeanDefinitionParser} from the register implementations using
 * the local name of the supplied {@link Element}.
 */
@Nullable
private BeanDefinitionParser findParserForElement(Element element, ParserContext parserContext) {
   String localName = parserContext.getDelegate().getLocalName(element);
   //对于<context:annotation-config/>返回的是AnnotationConfigBeanDefinitionParser，
   //对于<context:component-scan/>返回的是ComponentScanBeanDefinitionParser。
   BeanDefinitionParser parser = this.parsers.get(localName);
   if (parser == null) {
      parserContext.getReaderContext().fatal(
            "Cannot locate BeanDefinitionParser for element [" + localName + "]", element);
   }
   return parser;
}
```

则对于`<context:annotation-config/>`，最终调用的是AnnotationConfigBeanDefinitionParser的parse方法；对于`<context:component-scan/>`，最终调用的是ComponentScanBeanDefinitionParser的parse方法。

查看AnnotationConfigBeanDefinitionParser的parse方法

```java
public class AnnotationConfigBeanDefinitionParser implements BeanDefinitionParser {

   @Override
   @Nullable
   public BeanDefinition parse(Element element, ParserContext parserContext) {
      Object source = parserContext.extractSource(element);

      // Obtain bean definitions for all relevant BeanPostProcessors.
      //注意这一行，向BeanFcotry中注册相关注解的处理处理器，并返回这些处理器的定义
      Set<BeanDefinitionHolder> processorDefinitions =
            AnnotationConfigUtils.registerAnnotationConfigProcessors(parserContext.getRegistry(), source);

      // Register component for the surrounding <context:annotation-config> element.
      CompositeComponentDefinition compDefinition = new CompositeComponentDefinition(element.getTagName(), source);
      parserContext.pushContainingComponent(compDefinition);

      // Nest the concrete beans in the surrounding component.
      for (BeanDefinitionHolder processorDefinition : processorDefinitions) {
         parserContext.registerComponent(new BeanComponentDefinition(processorDefinition));
      }

      // Finally register the composite component.
      parserContext.popAndRegisterContainingComponent();

      return null;
   }

}
```

看一下上面说到的AnnotationConfigUtils.registerAnnotationConfigProcessors方法

```java
public static Set<BeanDefinitionHolder> registerAnnotationConfigProcessors(
      BeanDefinitionRegistry registry, @Nullable Object source) {

   DefaultListableBeanFactory beanFactory = unwrapDefaultListableBeanFactory(registry);
   if (beanFactory != null) {
      if (!(beanFactory.getDependencyComparator() instanceof AnnotationAwareOrderComparator)) {
         beanFactory.setDependencyComparator(AnnotationAwareOrderComparator.INSTANCE);
      }
      if (!(beanFactory.getAutowireCandidateResolver() instanceof ContextAnnotationAutowireCandidateResolver)) {
         beanFactory.setAutowireCandidateResolver(new ContextAnnotationAutowireCandidateResolver());
      }
   }

   Set<BeanDefinitionHolder> beanDefs = new LinkedHashSet<>(8);

   if (!registry.containsBeanDefinition(CONFIGURATION_ANNOTATION_PROCESSOR_BEAN_NAME)) {
      RootBeanDefinition def = new RootBeanDefinition(ConfigurationClassPostProcessor.class);
      def.setSource(source);
      beanDefs.add(registerPostProcessor(registry, def, CONFIGURATION_ANNOTATION_PROCESSOR_BEAN_NAME));
   }

   if (!registry.containsBeanDefinition(AUTOWIRED_ANNOTATION_PROCESSOR_BEAN_NAME)) {
      RootBeanDefinition def = new RootBeanDefinition(AutowiredAnnotationBeanPostProcessor.class);
      def.setSource(source);
      beanDefs.add(registerPostProcessor(registry, def, AUTOWIRED_ANNOTATION_PROCESSOR_BEAN_NAME));
   }

   // Check for JSR-250 support, and if present add the CommonAnnotationBeanPostProcessor.
   if (jsr250Present && !registry.containsBeanDefinition(COMMON_ANNOTATION_PROCESSOR_BEAN_NAME)) {
      RootBeanDefinition def = new RootBeanDefinition(CommonAnnotationBeanPostProcessor.class);
      def.setSource(source);
      beanDefs.add(registerPostProcessor(registry, def, COMMON_ANNOTATION_PROCESSOR_BEAN_NAME));
   }

   // Check for JPA support, and if present add the PersistenceAnnotationBeanPostProcessor.
   if (jpaPresent && !registry.containsBeanDefinition(PERSISTENCE_ANNOTATION_PROCESSOR_BEAN_NAME)) {
      RootBeanDefinition def = new RootBeanDefinition();
      try {
         def.setBeanClass(ClassUtils.forName(PERSISTENCE_ANNOTATION_PROCESSOR_CLASS_NAME,
               AnnotationConfigUtils.class.getClassLoader()));
      }
      catch (ClassNotFoundException ex) {
         throw new IllegalStateException(
               "Cannot load optional framework class: " + PERSISTENCE_ANNOTATION_PROCESSOR_CLASS_NAME, ex);
      }
      def.setSource(source);
      beanDefs.add(registerPostProcessor(registry, def, PERSISTENCE_ANNOTATION_PROCESSOR_BEAN_NAME));
   }

   if (!registry.containsBeanDefinition(EVENT_LISTENER_PROCESSOR_BEAN_NAME)) {
      RootBeanDefinition def = new RootBeanDefinition(EventListenerMethodProcessor.class);
      def.setSource(source);
      beanDefs.add(registerPostProcessor(registry, def, EVENT_LISTENER_PROCESSOR_BEAN_NAME));
   }

   if (!registry.containsBeanDefinition(EVENT_LISTENER_FACTORY_BEAN_NAME)) {
      RootBeanDefinition def = new RootBeanDefinition(DefaultEventListenerFactory.class);
      def.setSource(source);
      beanDefs.add(registerPostProcessor(registry, def, EVENT_LISTENER_FACTORY_BEAN_NAME));
   }

   return beanDefs;
}
```

可以确认，通过对`<context:annotation-config/>`的解析，的确向Spring容器中注册AutowiredAnnotationBeanPostProcessor、CommonAnnotationBeanPostProcessor、PersistenceAnnotationBeanPostProcessor 及 requiredAnnotationBeanPostProcessor 这 4 个 BeanPostProcessor

查看ComponentScanBeanDefinitionParser的parse方法

```java
@Override
@Nullable
public BeanDefinition parse(Element element, ParserContext parserContext) {
   String basePackage = element.getAttribute(BASE_PACKAGE_ATTRIBUTE);
   basePackage = parserContext.getReaderContext().getEnvironment().resolvePlaceholders(basePackage);
   String[] basePackages = StringUtils.tokenizeToStringArray(basePackage,
         ConfigurableApplicationContext.CONFIG_LOCATION_DELIMITERS);

   // Actually scan for bean definitions and register them.
   //配置扫描器
   ClassPathBeanDefinitionScanner scanner = configureScanner(parserContext, element);
   //扫描对应的包路径
   Set<BeanDefinitionHolder> beanDefinitions = scanner.doScan(basePackages);
   //注册到BeanFactory
   registerComponents(parserContext.getReaderContext(), beanDefinitions, element);

   return null;
}
```

跟进步骤里看，可以看到通过对`<context:component-scan/>` 配置项的解析处理，确实启用了对类包进行扫描以实施注释驱动 Bean 定义的功能，同时还启用了注释驱动自动注入的功能（即还隐式地在内部注册了 AutowiredAnnotationBeanPostProcessor 和 CommonAnnotationBeanPostProcessor）。

对于SpringMVC的`<mvc:annotation-driven/>`配置项，也可以找到对应的解析处理代码MvcNamespaceHandler，这里就不继续记录了。