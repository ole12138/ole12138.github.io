---
title: "Spring从xml启动注解扫描的过程"
date: 2021-01-23T15:06:16+08:00
draft: false
categories: 
- Spring
tags:
- Spring源码
---

# Spring从xml启动注解扫描的过程

## 问题引入

实际的Spring项目中，往往是：

- 在配置文件中开启注解自动扫描（还需要引入context命名空间xmlns:context以及标签格式说明context.xsd)，

  - ```xml
    <beans xmlns="http://www.springframework.org/schema/beans"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xmlns:context="http://www.springframework.org/schema/context"
           xsi:schemaLocation="<其他一些schema>
                               http://www.springframework.org/schema/context
                               http://www.springframework.org/schema/context/spring-context.xsd">
        <context:component-scan base-package="com.jingmin.*"/>
    </beans>
    ```

- 然后在相应的包下直接使用注解直接注入Bean。

  - ```java
    @Component
    class Xxx{
        ...
    }
    ```



那么，使用过程中，就会有这样的疑问：

- Spring是怎么实现读取context标签的？
- 读取到`context:component-scan`又是怎么开启注解扫描的？
- 扫描到注解之后又是怎么处理的？

## 源码分析

### AbstractApplicationContext#refresh

[前文](../Spring源码分析起步)介绍过，

- 不论是用`new ClassPathXmlApplicationContext("applicationContext.xml")`启动Spring，
- 还是在SpringMVC的web.xml配置监听器启动Spring，

最终都是调用AbstractApplicationContext的refresh()方法来完成Spring容器的启动和bean的管理的。

```java
public abstract class AbstractApplicationContext extends DefaultResourceLoader
implements ConfigurableApplicationContext {
	...
	@Override
	public void refresh() throws BeansException, IllegalStateException {
		synchronized (this.startupShutdownMonitor) {
			//调用容器准备刷新的方法，获取容器的当时时间，同时给容器设置同步标识 
			prepareRefresh();
 
			//告诉子类启动refreshBeanFactory()方法，Bean定义资源文件的载入从  
            //子类的refreshBeanFactory()方法启动  
			ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();
 
			//为BeanFactory配置容器特性，例如类加载器、事件处理器等 
			prepareBeanFactory(beanFactory);
 
			try {
				//为容器的某些子类指定特殊的BeanPost事件处理器 
				postProcessBeanFactory(beanFactory);
 
				//调用所有注册的BeanFactoryPostProcessor的Bean  
				invokeBeanFactoryPostProcessors(beanFactory);
 
				//为BeanFactory注册BeanPost事件处理器.  
                //BeanPostProcessor是Bean后置处理器，用于监听容器触发的事件
				registerBeanPostProcessors(beanFactory);
 
				//初始化信息源，和国际化相关.  
				initMessageSource();
 
				//初始化容器事件传播器.  
				initApplicationEventMulticaster();
 
				//调用子类的某些特殊Bean初始化方法
				onRefresh();
 
				//为事件传播器注册事件监听器.  
				registerListeners();
 
				 //初始化所有剩余的单态Bean.  
				finishBeanFactoryInitialization(beanFactory);
 
				//初始化容器的生命周期事件处理器，并发布容器的生命周期事件  
				finishRefresh();
			}
			catch (BeansException ex) {
				if (logger.isWarnEnabled()) {
					logger.warn("Exception encountered during context initialization - " +
							"cancelling refresh attempt: " + ex);
				}
				//销毁以创建的单态Bean  
				destroyBeans();
				//取消refresh操作，重置容器的同步标识. 
				cancelRefresh(ex);
				// Propagate exception to caller.
				throw ex;
			}
			finally {
				resetCommonCaches();
			}
		}
	}
}
```

Spring根据配置文件，将`<bean>`标签中的声明转化为BeanDefinition的过程，是在上面的这一调用中完成的：
`ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();`

那么我们猜想，context标签的解析也应该在这一步完成。

### AbstractApplicationContext#obtainFreshBeanFactory

```java
protected ConfigurableListableBeanFactory obtainFreshBeanFactory() {
    refreshBeanFactory();
    return getBeanFactory();
}
```

跟一下其中的refreshBeanFactory()方法，这个方法在子类AbstractRefreshableApplicationContext中实现。

事实上，上面AbstractApplicationContext#refresh方法也是其子类调用的，所以在调用refreshBeanFactory方法时调用的是子类覆写后的refreshBeanFactory()方法

### AbstractRefreshableApplicationContext#refreshBeanFactory

```java
@Override
protected final void refreshBeanFactory() throws BeansException {
   if (hasBeanFactory()) {
      destroyBeans();
      closeBeanFactory();
   }
   try {
      //创建BeanFactory
      DefaultListableBeanFactory beanFactory = createBeanFactory();
      beanFactory.setSerializationId(getId());
      customizeBeanFactory(beanFactory);
      //将声明的bean标签，转换为BeanDefinition，并放入BeanFactory
      loadBeanDefinitions(beanFactory);
      this.beanFactory = beanFactory;
   }
   catch (IOException ex) {
      throw new ApplicationContextException("I/O error parsing bean definition source for " + getDisplayName(), ex);
   }
}
```

跟一下`loadBeanDefinitions(beanFactory);`。

loadBeanDefinitions方法也是被子类AbstractXmlApplicationContext覆写了，实际调用也是子类在调用。

### AbstractXmlApplicationContext#loadBeanDefinitions(DefaultListableBeanFactory beanFactory)

```java
@Override
protected void loadBeanDefinitions(DefaultListableBeanFactory beanFactory) throws BeansException, IOException {
   // Create a new XmlBeanDefinitionReader for the given BeanFactory.
   XmlBeanDefinitionReader beanDefinitionReader = new XmlBeanDefinitionReader(beanFactory);

   // Configure the bean definition reader with this context's
   // resource loading environment.
   beanDefinitionReader.setEnvironment(this.getEnvironment());
   beanDefinitionReader.setResourceLoader(this);
   beanDefinitionReader.setEntityResolver(new ResourceEntityResolver(this));

   // Allow a subclass to provide custom initialization of the reader,
   // then proceed with actually loading the bean definitions.
   initBeanDefinitionReader(beanDefinitionReader);
   //实际加载bean标签为BeanDefinition是在这里完成的
   loadBeanDefinitions(beanDefinitionReader);
}
```

跟一下`loadBeanDefinitions(beanDefinitionReader);`.

### AbstractXmlApplicationContext#loadBeanDefinitions(XmlBeanDefinitionReader reader)

```java
protected void loadBeanDefinitions(XmlBeanDefinitionReader reader) throws BeansException, IOException {
   //getConfigResources方法在ClassPathXmlApplicationContext中被覆写掉了,会将applictionContext.xml包装为Resource
   Resource[] configResources = getConfigResources();
   if (configResources != null) {
      //实际解析配置文件为BeanDefinition的过程
      reader.loadBeanDefinitions(configResources);
   }
   String[] configLocations = getConfigLocations();
   if (configLocations != null) {
      reader.loadBeanDefinitions(configLocations);
   }
}
```

跟一下`reader.loadBeanDefinitions(configResources);`

loadBeanDefinitions这个方法在XmlBeanDefinitionReader这个类的父类AbstractBeanDefinitionReader中定义。

### AbstractBeanDefinitionReader#loadBeanDefinitions(Resource... resources)

```
@Override
public int loadBeanDefinitions(Resource... resources) throws BeanDefinitionStoreException {
   Assert.notNull(resources, "Resource array must not be null");
   int count = 0;
   for (Resource resource : resources) {
   	  //分别加载每个配置文件
      count += loadBeanDefinitions(resource);
   }
   return count;
}
```

跟一下`count += loadBeanDefinitions(resource);`

回到XmlBeanDefinitionReader类中，这个方法已经被覆写掉了。

### XmlBeanDefinitionReader#loadBeanDefinitions(Resource resource)

```java
@Override
public int loadBeanDefinitions(Resource resource) throws BeanDefinitionStoreException {
   return loadBeanDefinitions(new EncodedResource(resource));
}
```

跟一下`return loadBeanDefinitions(new EncodedResource(resource));`

### XmlBeanDefinitionReader#loadBeanDefinitions(EncodedResource encodedResource)

```java
public int loadBeanDefinitions(EncodedResource encodedResource) throws BeanDefinitionStoreException {
   Assert.notNull(encodedResource, "EncodedResource must not be null");
   if (logger.isTraceEnabled()) {
      logger.trace("Loading XML bean definitions from " + encodedResource);
   }

   Set<EncodedResource> currentResources = this.resourcesCurrentlyBeingLoaded.get();

   if (!currentResources.add(encodedResource)) {
      throw new BeanDefinitionStoreException(
            "Detected cyclic loading of " + encodedResource + " - check your import definitions!");
   }

   try (InputStream inputStream = encodedResource.getResource().getInputStream()) {
      InputSource inputSource = new InputSource(inputStream);
      if (encodedResource.getEncoding() != null) {
         inputSource.setEncoding(encodedResource.getEncoding());
      }
      //实际解析配置文件为BeanDefinition的过程
      return doLoadBeanDefinitions(inputSource, encodedResource.getResource());
   }
   catch (IOException ex) {
      throw new BeanDefinitionStoreException(
            "IOException parsing XML document from " + encodedResource.getResource(), ex);
   }
   finally {
      currentResources.remove(encodedResource);
      if (currentResources.isEmpty()) {
         this.resourcesCurrentlyBeingLoaded.remove();
      }
   }
}
```

跟一下`return doLoadBeanDefinitions(inputSource, encodedResource.getResource());`

### XmlBeanDefinitionReader#doLoadBeanDefinitions

```java
protected int doLoadBeanDefinitions(InputSource inputSource, Resource resource)
      throws BeanDefinitionStoreException {

   try {
   	  //xml解析为document文档
      Document doc = doLoadDocument(inputSource, resource);
      //从文档树上解析元素，存储为BeanDefinition，并注册到Spring工厂中
      int count = registerBeanDefinitions(doc, resource);
      if (logger.isDebugEnabled()) {
         logger.debug("Loaded " + count + " bean definitions from " + resource);
      }
      return count;
   }
   catch (BeanDefinitionStoreException ex) {
      throw ex;
   }
   catch (SAXParseException ex) {
      throw new XmlBeanDefinitionStoreException(resource.getDescription(),
            "Line " + ex.getLineNumber() + " in XML document from " + resource + " is invalid", ex);
   }
   catch (SAXException ex) {
      throw new XmlBeanDefinitionStoreException(resource.getDescription(),
            "XML document from " + resource + " is invalid", ex);
   }
   catch (ParserConfigurationException ex) {
      throw new BeanDefinitionStoreException(resource.getDescription(),
            "Parser configuration exception parsing XML from " + resource, ex);
   }
   catch (IOException ex) {
      throw new BeanDefinitionStoreException(resource.getDescription(),
            "IOException parsing XML document from " + resource, ex);
   }
   catch (Throwable ex) {
      throw new BeanDefinitionStoreException(resource.getDescription(),
            "Unexpected exception parsing XML document from " + resource, ex);
   }
}
```

跟一下`int count = registerBeanDefinitions(doc, resource);`

### XmlBeanDefinitionReader#registerBeanDefinitions



```java
public int registerBeanDefinitions(Document doc, Resource resource) throws BeanDefinitionStoreException {
   BeanDefinitionDocumentReader documentReader = createBeanDefinitionDocumentReader();
   int countBefore = getRegistry().getBeanDefinitionCount();
   //将document文档中的元素转化为BeanDefinition(s)，并注册到工厂中
   documentReader.registerBeanDefinitions(doc, createReaderContext(resource));
   return getRegistry().getBeanDefinitionCount() - countBefore;
}
```

想要跟一下`documentReader.registerBeanDefinitions(doc, createReaderContext(resource));`

但是发现BeanDefinitionDocumentReader中只定义了空实现的registerBeanDefinitions方法，那么一定是通过子类覆写了。

所以这里还需要跟一下前一个步骤`BeanDefinitionDocumentReader documentReader = createBeanDefinitionDocumentReader();`

```java
protected BeanDefinitionDocumentReader createBeanDefinitionDocumentReader() {
   return BeanUtils.instantiateClass(this.documentReaderClass);
}
```

继续跟去看一下`this.documentReaderClass`是什么

```java
private Class<? extends BeanDefinitionDocumentReader> documentReaderClass =
      DefaultBeanDefinitionDocumentReader.class;
```

则说明`documentReader.registerBeanDefinitions(doc, createReaderContext(resource));`使用的是DefaultBeanDefinitionDocumentReader中的registerBeanDefinitions方法，跟进去看看。

### DefaultBeanDefinitionDocumentReader#registerBeanDefinitions

```java
@Override
public void registerBeanDefinitions(Document doc, XmlReaderContext readerContext) {
   this.readerContext = readerContext;
   doRegisterBeanDefinitions(doc.getDocumentElement());
}
```

继续跟`doRegisterBeanDefinitions(doc.getDocumentElement());`

### DefaultBeanDefinitionDocumentReader#doRegisterBeanDefinitions

```java
@SuppressWarnings("deprecation")  // for Environment.acceptsProfiles(String...)
protected void doRegisterBeanDefinitions(Element root) {
   // Any nested <beans> elements will cause recursion in this method.
   BeanDefinitionParserDelegate parent = this.delegate;
   this.delegate = createDelegate(getReaderContext(), root, parent);

   if (this.delegate.isDefaultNamespace(root)) {
      String profileSpec = root.getAttribute(PROFILE_ATTRIBUTE);
      if (StringUtils.hasText(profileSpec)) {
         String[] specifiedProfiles = StringUtils.tokenizeToStringArray(
               profileSpec, BeanDefinitionParserDelegate.MULTI_VALUE_ATTRIBUTE_DELIMITERS);
         // We cannot use Profiles.of(...) since profile expressions are not supported
         // in XML config. See SPR-12458 for details.
         if (!getReaderContext().getEnvironment().acceptsProfiles(specifiedProfiles)) {
            if (logger.isDebugEnabled()) {
               logger.debug("Skipped XML bean definition file due to specified profiles [" + profileSpec +
                     "] not matching: " + getReaderContext().getResource());
            }
            return;
         }
      }
   }

   preProcessXml(root);
   //解析document为Definitions
   parseBeanDefinitions(root, this.delegate);
   postProcessXml(root);

   this.delegate = parent;
}
```

继续跟`parseBeanDefinitions(root, this.delegate);`

### DefaultBeanDefinitionDocumentReader#parseBeanDefinitions

```java
protected void parseBeanDefinitions(Element root, BeanDefinitionParserDelegate delegate) {
   if (delegate.isDefaultNamespace(root)) {
      NodeList nl = root.getChildNodes();
      for (int i = 0; i < nl.getLength(); i++) {
         Node node = nl.item(i);
         if (node instanceof Element) {
            Element ele = (Element) node;
            if (delegate.isDefaultNamespace(ele)) {
               //解析默认标签：import,alias,bean, beans
               //将bean标签解析为DeanDefinition也在这一步完成
               parseDefaultElement(ele, delegate);
            }
            else {
               //解析自定义标签：context，aop等
               delegate.parseCustomElement(ele);
            }
         }
      }
   }
   else {
       //解析自定义标签：context，aop等
      delegate.parseCustomElement(root);
   }
}
```

我们关注的是自动扫描配置`<context:component-scan base-package="com.jingmin.*"/>`是如何启动的，
所以要跟一下`delegate.parseCustomElement(root);`或`delegate.parseCustomElement(ele);`

### BeanDefinitionParserDelegate#parseCustomElement

```java
@Nullable
public BeanDefinition parseCustomElement(Element ele) {
   return parseCustomElement(ele, null);
}
```

继续跟`return parseCustomElement(ele, null);`

```java
@Nullable
public BeanDefinition parseCustomElement(Element ele, @Nullable BeanDefinition containingBd) {
   String namespaceUri = getNamespaceURI(ele);
   if (namespaceUri == null) {
      return null;
   }
   //获取解析器，根据标签命名空间，比如context，进一步找到对应的处理器
   NamespaceHandler handler = this.readerContext.getNamespaceHandlerResolver().resolve(namespaceUri);
   if (handler == null) {
      error("Unable to locate Spring NamespaceHandler for XML schema namespace [" + namespaceUri + "]", ele);
      return null;
   }
   //使用对应的处理器，根据标签内容的指示，完成指定的功能
   return handler.parse(ele, new ParserContext(this.readerContext, this, containingBd));
}
```

跟一下`NamespaceHandler handler = this.readerContext.getNamespaceHandlerResolver().resolve(namespaceUri);`

这个分析过程稍稍有点曲折，下面两节都在跟这里的代码。

### XmlReaderContext#getNamespaceHandlerResolver

先看一下其中的getNamespaceHandlerResolver方法

```java
public final NamespaceHandlerResolver getNamespaceHandlerResolver() {
   return this.namespaceHandlerResolver;
}
```

再看一下这个属性是怎么设置的：

```java
public XmlReaderContext(
      Resource resource, ProblemReporter problemReporter,
      ReaderEventListener eventListener, SourceExtractor sourceExtractor,
      XmlBeanDefinitionReader reader, NamespaceHandlerResolver namespaceHandlerResolver) {

   super(resource, problemReporter, eventListener, sourceExtractor);
   this.reader = reader;
   //在构造体中完成的初始化
   this.namespaceHandlerResolver = namespaceHandlerResolver;
}
```

而它 又是被XmlBeanDefinitionReader中的方法调用的：
(实际上，前面分析中，在XmlBeanDefinitionReader的registerBeanDefinitions方法运行的过程中，已经调用过这个方法了)

```java
public XmlReaderContext createReaderContext(Resource resource) {
   return new XmlReaderContext(resource, this.problemReporter, this.eventListener,
         this.sourceExtractor, this, getNamespaceHandlerResolver());
}
```

进一步跟一下其中的`getNamespaceHandlerResolver()`方法

```java
public NamespaceHandlerResolver getNamespaceHandlerResolver() {
   if (this.namespaceHandlerResolver == null) {
      this.namespaceHandlerResolver = createDefaultNamespaceHandlerResolver();
   }
   return this.namespaceHandlerResolver;
}
```

继续跟`this.namespaceHandlerResolver = createDefaultNamespaceHandlerResolver();`

```java
protected NamespaceHandlerResolver createDefaultNamespaceHandlerResolver() {
   ClassLoader cl = (getResourceLoader() != null ? getResourceLoader().getClassLoader() : getBeanClassLoader());
   return new DefaultNamespaceHandlerResolver(cl);
}
```

最终我们得知`NamespaceHandler handler = this.readerContext.getNamespaceHandlerResolver().resolve(namespaceUri);`中`getNamespaceHandlerResolver()`过程获取的是`DefaultNamespaceHandlerResolver(cl);`的实例。

接下来，我们要跟一下DefaultNamespaceHandlerResolver的resolve方法。

### DefaultNamespaceHandlerResolver#resolve

```java
@Override
@Nullable
public NamespaceHandler resolve(String namespaceUri) {
   //获取当前resolver中存在的handlermapping
   Map<String, Object> handlerMappings = getHandlerMappings();
   //找命名空间namespaceUri对应的handler名称或实例
   Object handlerOrClassName = handlerMappings.get(namespaceUri);
   if (handlerOrClassName == null) {
   	  //找不到handler
      return null;
   }
   else if (handlerOrClassName instanceof NamespaceHandler) {
      //如果存的是handler的实例，返回handler实例
      return (NamespaceHandler) handlerOrClassName;
   }
   else {
      //如果存的是handler的名称，实例化
      String className = (String) handlerOrClassName;
      try {
         Class<?> handlerClass = ClassUtils.forName(className, this.classLoader);
         if (!NamespaceHandler.class.isAssignableFrom(handlerClass)) {
            throw new FatalBeanException("Class [" + className + "] for namespace [" + namespaceUri +
                  "] does not implement the [" + NamespaceHandler.class.getName() + "] interface");
         }
         //反射实例化
         NamespaceHandler namespaceHandler = (NamespaceHandler) BeanUtils.instantiateClass(handlerClass);
         //handler初始化工作，一般是注册命名空间的属性
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

先跟一下`Map<String, Object> handlerMappings = getHandlerMappings();`

### DefaultNamespaceHandlerResolver#getHandlerMappings

```java
private Map<String, Object> getHandlerMappings() {
   //从本实例的属性中加载handlerMappings
   Map<String, Object> handlerMappings = this.handlerMappings;
   if (handlerMappings == null) {
      synchronized (this) {
         handlerMappings = this.handlerMappings;
         if (handlerMappings == null) {
            //如果还没初始化handlerMappings
            if (logger.isTraceEnabled()) {
               logger.trace("Loading NamespaceHandler mappings from [" + this.handlerMappingsLocation + "]");
            }
            try {
               //从配置文件中加载handlerMappings的（配置文件中存的是：命名空间-handler类名键值对)	
               Properties mappings =
                     PropertiesLoaderUtils.loadAllProperties(this.handlerMappingsLocation, this.classLoader);
               if (logger.isTraceEnabled()) {
                  logger.trace("Loaded NamespaceHandler mappings: " + mappings);
               }
               handlerMappings = new ConcurrentHashMap<>(mappings.size());
               CollectionUtils.mergePropertiesIntoMap(mappings, handlerMappings);
               this.handlerMappings = handlerMappings;
            }
            catch (IOException ex) {
               throw new IllegalStateException(
                     "Unable to load NamespaceHandler mappings from location [" + this.handlerMappingsLocation + "]", ex);
            }
         }
      }
   }
   return handlerMappings;
}
```

跟一下，还没初始化handlerMappings时，`Properties mappings = PropertiesLoaderUtils.loadAllProperties(this.handlerMappingsLocation, this.classLoader);`的过程。

主要是看一下这里的`this.handlerMappingsLocation`属性存的是什么。

在这里：

```java
public DefaultNamespaceHandlerResolver(@Nullable ClassLoader classLoader, String handlerMappingsLocation) {
   Assert.notNull(handlerMappingsLocation, "Handler mappings location must not be null");
   this.classLoader = (classLoader != null ? classLoader : ClassUtils.getDefaultClassLoader());
   //通过constructor传入
   this.handlerMappingsLocation = handlerMappingsLocation;
}
```

它又会被另一个constructor调用：（前面在XmlBeanDefinitionReader#createDefaultNamespaceHandlerResolver方法中已经调用过了)

```java
public DefaultNamespaceHandlerResolver() {
   this(null, DEFAULT_HANDLER_MAPPINGS_LOCATION);
}
```

继续看一下`DEFAULT_HANDLER_MAPPINGS_LOCATION`存的是什么

```java
public static final String DEFAULT_HANDLER_MAPPINGS_LOCATION = "META-INF/spring.handlers";
```

在spring-context-x.x.x.jar中的可以找到\META-INF\spring.handlers文件，以下是文件内容：

```
http\://www.springframework.org/schema/context=org.springframework.context.config.ContextNamespaceHandler
http\://www.springframework.org/schema/jee=org.springframework.ejb.config.JeeNamespaceHandler
http\://www.springframework.org/schema/lang=org.springframework.scripting.config.LangNamespaceHandler
http\://www.springframework.org/schema/task=org.springframework.scheduling.config.TaskNamespaceHandler
http\://www.springframework.org/schema/cache=org.springframework.cache.config.CacheNamespaceHandler
```

可以看到命名空间`http\://www.springframework.org/schema/context`对应的handler类名是`org.springframework.context.config.ContextNamespaceHandler`

再返回DefaultNamespaceHandlerResolver#getHandlerMappings方法中去看，
还没初始化handlerMappings时，`Properties mappings = PropertiesLoaderUtils.loadAllProperties(this.handlerMappingsLocation, this.classLoader);`会加载META-INF\spring.handlers文件，作为handlermappings

再返回到DefaultNamespaceHandlerResolver#resolve方法中去看：

```java
...
//如果存的是handler的名称，实例化
String className = (String) handlerOrClassName;
try {
    Class<?> handlerClass = ClassUtils.forName(className, this.classLoader);
    if (!NamespaceHandler.class.isAssignableFrom(handlerClass)) {
        throw new FatalBeanException("Class [" + className + "] for namespace [" + namespaceUri +
                                     "] does not implement the [" + NamespaceHandler.class.getName() + "] interface");
    }
    //反射实例化
    NamespaceHandler namespaceHandler = (NamespaceHandler) BeanUtils.instantiateClass(handlerClass);
    //handler初始化工作，一般是注册命名空间的属性
    namespaceHandler.init();
    handlerMappings.put(namespaceUri, namespaceHandler);
    return namespaceHandler;
}catch(...){..}
```



从spring.handlers文件中可以看到命名空间`http\://www.springframework.org/schema/context`对应的handler类名是`org.springframework.context.config.ContextNamespaceHandler`，
后面`return namespaceHandler;`返回的是ContextNamespaceHandler的实例

这里跟一下`namespaceHandler.init();`，实际上跟的是ContextNamespaceHandler的init方法

### ContextNamespaceHandler#init

```java
@Override
public void init() {
   registerBeanDefinitionParser("property-placeholder", new PropertyPlaceholderBeanDefinitionParser());
   registerBeanDefinitionParser("property-override", new PropertyOverrideBeanDefinitionParser());
   registerBeanDefinitionParser("annotation-config", new AnnotationConfigBeanDefinitionParser());
   //注册处理context:component-scan的解析器
   registerBeanDefinitionParser("component-scan", new ComponentScanBeanDefinitionParser());
   registerBeanDefinitionParser("load-time-weaver", new LoadTimeWeaverBeanDefinitionParser());
   registerBeanDefinitionParser("spring-configured", new SpringConfiguredBeanDefinitionParser());
   registerBeanDefinitionParser("mbean-export", new MBeanExportBeanDefinitionParser());
   registerBeanDefinitionParser("mbean-server", new MBeanServerBeanDefinitionParser());
}
```

再回到BeanDefinitionParserDelegate#parseCustomElement中去看：

```java
@Nullable
public BeanDefinition parseCustomElement(Element ele, @Nullable BeanDefinition containingBd) {
   String namespaceUri = getNamespaceURI(ele);
   if (namespaceUri == null) {
      return null;
   }
   //获取解析器，根据标签命名空间，比如context，进一步找到对应的处理器
   NamespaceHandler handler = this.readerContext.getNamespaceHandlerResolver().resolve(namespaceUri);
   if (handler == null) {
      error("Unable to locate Spring NamespaceHandler for XML schema namespace [" + namespaceUri + "]", ele);
      return null;
   }
   //使用对应的处理器，根据标签内容的指示，完成指定的功能
   return handler.parse(ele, new ParserContext(this.readerContext, this, containingBd));
}
```

前面完整的分析了`NamespaceHandler handler = this.readerContext.getNamespaceHandlerResolver().resolve(namespaceUri);`过程，对于context命名空间，最终返回的是ContextNamespaceHandler的实例

这里再跟一下`return handler.parse(ele, new ParserContext(this.readerContext, this, containingBd));`
实际上跟的是ContextNamespaceHandler的parse方法。
而ContextNamespaceHandler直接继承了父类NamespaceHandlerSupport的parse方法。

### NamespaceHandlerSupport#parse

```java
@Override
@Nullable
public BeanDefinition parse(Element element, ParserContext parserContext) {
   BeanDefinitionParser parser = findParserForElement(element, parserContext);
   return (parser != null ? parser.parse(element, parserContext) : null);
}
```

跟一下`BeanDefinitionParser parser = findParserForElement(element, parserContext);`

```java
private BeanDefinitionParser findParserForElement(Element element, ParserContext parserContext) {
   String localName = parserContext.getDelegate().getLocalName(element);
   //根据属性名获取对应的parser
   BeanDefinitionParser parser = this.parsers.get(localName);
   if (parser == null) {
      parserContext.getReaderContext().fatal(
            "Cannot locate BeanDefinitionParser for element [" + localName + "]", element);
   }
   return parser;
}
```

这里跟一下`BeanDefinitionParser parser = this.parsers.get(localName);`中的this.parsers，看看它存了什么

然后发现只有一个方法可以修改this.parsers

```java
protected final void registerBeanDefinitionParser(String elementName, BeanDefinitionParser parser) {
		this.parsers.put(elementName, parser);
	}
```

前面ContextNamespaceHandler#init中我们调用过这个方法

```java
@Override
public void init() {
   ...
   //注册处理context:component-scan的解析器
   registerBeanDefinitionParser("component-scan", new ComponentScanBeanDefinitionParser());
   ...
}
```

再回到NamespaceHandlerSupport#parse去看，就可以知道，
对于Spring配置文件中的`<context:component-scan base-package="com.jingmin.*"/>`语句，
最终调用的ComponentScanBeanDefinitionParser的parse方法。跟一下。

### ComponentScanBeanDefinitionParser#parse

```java
public BeanDefinition parse(Element element, ParserContext parserContext) {
   //获取<context:component-scan base-package="com.jingmin.*"/>中的base-package属性值
   String basePackage = element.getAttribute(BASE_PACKAGE_ATTRIBUTE);
   //解析${xxx}占位符
   basePackage = parserContext.getReaderContext().getEnvironment().resolvePlaceholders(basePackage);
   //可能有多个位置需要扫描，根据分割符分成数组
   String[] basePackages = StringUtils.tokenizeToStringArray(basePackage,
         ConfigurableApplicationContext.CONFIG_LOCATION_DELIMITERS);

   // Actually scan for bean definitions and register them.
   ClassPathBeanDefinitionScanner scanner = configureScanner(parserContext, element);
   Set<BeanDefinitionHolder> beanDefinitions = scanner.doScan(basePackages);
   registerComponents(parserContext.getReaderContext(), beanDefinitions, element);

   return null;
}
```

我们需要跟一下`ClassPathBeanDefinitionScanner scanner = configureScanner(parserContext, element);`和`Set<BeanDefinitionHolder> beanDefinitions = scanner.doScan(basePackages);`

### ComponentScanBeanDefinitionParser#configureScanner

```java
private static final String USE_DEFAULT_FILTERS_ATTRIBUTE = "use-default-filters";

protected ClassPathBeanDefinitionScanner configureScanner(ParserContext parserContext, Element element) {
   ///默认这个标志位是true
   boolean useDefaultFilters = true;
   //假如配置文件中是这样配置自动扫描的：
   //<context:component-scan base-package="com.jingmin.*"/>
   //这里没有设置use-default-filters属性，所以下面的判断会被跳过
   if (element.hasAttribute(USE_DEFAULT_FILTERS_ATTRIBUTE)) {
      useDefaultFilters = Boolean.parseBoolean(element.getAttribute(USE_DEFAULT_FILTERS_ATTRIBUTE));
   }
	//使用默认filter设置扫描器
   // Delegate bean definition registration to scanner class.
   ClassPathBeanDefinitionScanner scanner = createScanner(parserContext.getReaderContext(), useDefaultFilters);
   scanner.setBeanDefinitionDefaults(parserContext.getDelegate().getBeanDefinitionDefaults());
   scanner.setAutowireCandidatePatterns(parserContext.getDelegate().getAutowireCandidatePatterns());

   if (element.hasAttribute(RESOURCE_PATTERN_ATTRIBUTE)) {
      scanner.setResourcePattern(element.getAttribute(RESOURCE_PATTERN_ATTRIBUTE));
   }

   try {
      parseBeanNameGenerator(element, scanner);
   }
   catch (Exception ex) {
      parserContext.getReaderContext().error(ex.getMessage(), parserContext.extractSource(element), ex.getCause());
   }

   try {
      parseScope(element, scanner);
   }
   catch (Exception ex) {
      parserContext.getReaderContext().error(ex.getMessage(), parserContext.extractSource(element), ex.getCause());
   }

   parseTypeFilters(element, scanner, parserContext);

   return scanner;
}
```

跟一下`ClassPathBeanDefinitionScanner scanner = createScanner(parserContext.getReaderContext(), useDefaultFilters);`

```java
protected ClassPathBeanDefinitionScanner createScanner(XmlReaderContext readerContext, boolean useDefaultFilters) {
   return new ClassPathBeanDefinitionScanner(readerContext.getRegistry(), useDefaultFilters,
         readerContext.getEnvironment(), readerContext.getResourceLoader());
}
public ClassPathBeanDefinitionScanner(BeanDefinitionRegistry registry, boolean useDefaultFilters,
			Environment environment, @Nullable ResourceLoader resourceLoader) {

		Assert.notNull(registry, "BeanDefinitionRegistry must not be null");
		this.registry = registry;

		if (useDefaultFilters) {
			registerDefaultFilters();
		}
		setEnvironment(environment);
		setResourceLoader(resourceLoader);
	}
```

跟一下其中的`registerDefaultFilters();`

这个方法不在ComponentScanBeanDefinitionParser中定义，而是直接继承ClassPathScanningCandidateComponentProvider而来。

### ClassPathScanningCandidateComponentProvider#registerDefaultFilters

```java
@SuppressWarnings("unchecked")
protected void registerDefaultFilters() {
   //看到这里添加了对@Component注解的扫描
   this.includeFilters.add(new AnnotationTypeFilter(Component.class));
   ClassLoader cl = ClassPathScanningCandidateComponentProvider.class.getClassLoader();
   try {
      this.includeFilters.add(new AnnotationTypeFilter(
            ((Class<? extends Annotation>) ClassUtils.forName("javax.annotation.ManagedBean", cl)), false));
      logger.trace("JSR-250 'javax.annotation.ManagedBean' found and supported for component scanning");
   }
   catch (ClassNotFoundException ex) {
      // JSR-250 1.1 API (as included in Java EE 6) not available - simply skip.
   }
   try {
      this.includeFilters.add(new AnnotationTypeFilter(
            ((Class<? extends Annotation>) ClassUtils.forName("javax.inject.Named", cl)), false));
      logger.trace("JSR-330 'javax.inject.Named' annotation found and supported for component scanning");
   }
   catch (ClassNotFoundException ex) {
      // JSR-330 API not available - simply skip.
   }
}
```

可以看到在这个方法中添加了对@Component注解的过滤

### ClassPathBeanDefinitionScanner#doScan

再回到ComponentScanBeanDefinitionParser#parse方法中，跟一下和`Set<BeanDefinitionHolder> beanDefinitions = scanner.doScan(basePackages);`

```java
protected Set<BeanDefinitionHolder> doScan(String... basePackages) {
   Assert.notEmpty(basePackages, "At least one base package must be specified");
   Set<BeanDefinitionHolder> beanDefinitions = new LinkedHashSet<>();
   for (String basePackage : basePackages) {
      //过滤出带@Component注解的类，并包装成BeanDefinition，用set返回
      Set<BeanDefinition> candidates = findCandidateComponents(basePackage);
      for (BeanDefinition candidate : candidates) {
         ScopeMetadata scopeMetadata = this.scopeMetadataResolver.resolveScopeMetadata(candidate);
         candidate.setScope(scopeMetadata.getScopeName());
         String beanName = this.beanNameGenerator.generateBeanName(candidate, this.registry);
         if (candidate instanceof AbstractBeanDefinition) {
            postProcessBeanDefinition((AbstractBeanDefinition) candidate, beanName);
         }
         if (candidate instanceof AnnotatedBeanDefinition) {
            AnnotationConfigUtils.processCommonDefinitionAnnotations((AnnotatedBeanDefinition) candidate);
         }
         if (checkCandidate(beanName, candidate)) {
            BeanDefinitionHolder definitionHolder = new BeanDefinitionHolder(candidate, beanName);
            definitionHolder =
                  AnnotationConfigUtils.applyScopedProxyMode(scopeMetadata, definitionHolder, this.registry);
            beanDefinitions.add(definitionHolder);
            registerBeanDefinition(definitionHolder, this.registry);
         }
      }
   }
   return beanDefinitions;
}
```

跟一下`Set<BeanDefinition> candidates = findCandidateComponents(basePackage);`

这个方法在ClassPathBeanDefinitionScanner的父类ClassPathScanningCandidateComponentProvider中定义。

### ClassPathScanningCandidateComponentProvider#findCandidateComponents

```java
public Set<BeanDefinition> findCandidateComponents(String basePackage) {
   if (this.componentsIndex != null && indexSupportsIncludeFilters()) {
      return addCandidateComponentsFromIndex(this.componentsIndex, basePackage);
   }
   else {
      return scanCandidateComponents(basePackage);
   }
}
```

跟一下`return scanCandidateComponents(basePackage);`

### ClassPathScanningCandidateComponentProvider#scanCandidateComponents

```java
private Set<BeanDefinition> scanCandidateComponents(String basePackage) {
   Set<BeanDefinition> candidates = new LinkedHashSet<>();
   try {
      String packageSearchPath = ResourcePatternResolver.CLASSPATH_ALL_URL_PREFIX +
            resolveBasePackage(basePackage) + '/' + this.resourcePattern;
      Resource[] resources = getResourcePatternResolver().getResources(packageSearchPath);
      boolean traceEnabled = logger.isTraceEnabled();
      boolean debugEnabled = logger.isDebugEnabled();
      for (Resource resource : resources) {
         if (traceEnabled) {
            logger.trace("Scanning " + resource);
         }
         if (resource.isReadable()) {
            try {
               MetadataReader metadataReader = getMetadataReaderFactory().getMetadataReader(resource);
               //判断是否是Component
               if (isCandidateComponent(metadataReader)) {
                  ScannedGenericBeanDefinition sbd = new ScannedGenericBeanDefinition(metadataReader);
                  sbd.setSource(resource);
                  if (isCandidateComponent(sbd)) {
                     if (debugEnabled) {
                        logger.debug("Identified candidate component class: " + resource);
                     }
                     candidates.add(sbd);
                  }
                  else {
                     if (debugEnabled) {
                        logger.debug("Ignored because not a concrete top-level class: " + resource);
                     }
                  }
               }
               else {
                  if (traceEnabled) {
                     logger.trace("Ignored because not matching any filter: " + resource);
                  }
               }
            }
            catch (Throwable ex) {
               throw new BeanDefinitionStoreException(
                     "Failed to read candidate component class: " + resource, ex);
            }
         }
         else {
            if (traceEnabled) {
               logger.trace("Ignored because not readable: " + resource);
            }
         }
      }
   }
   catch (IOException ex) {
      throw new BeanDefinitionStoreException("I/O failure during classpath scanning", ex);
   }
   return candidates;
}
```

跟一下`if (isCandidateComponent(metadataReader)) `语句中的isCandidateComponent方法

### ClassPathScanningCandidateComponentProvider#isCandidateComponent

```java
protected boolean isCandidateComponent(MetadataReader metadataReader) throws IOException {
   for (TypeFilter tf : this.excludeFilters) {
      if (tf.match(metadataReader, getMetadataReaderFactory())) {
         return false;
      }
   }
   for (TypeFilter tf : this.includeFilters) {
      if (tf.match(metadataReader, getMetadataReaderFactory())) {
         return isConditionMatch(metadataReader);
      }
   }
   return false;
}
```

前面在ClassPathScanningCandidateComponentProvider#registerDefaultFilters方法中有一行`this.includeFilters.add(new AnnotationTypeFilter(Component.class));`，表示向includeFilters中添加了对@Component注解的过滤，
在当前ClassPathScanningCandidateComponentProvider#isCandidateComponent方法中，就会将携带了@Component注解的类给过滤出来

一直返回去到ComponentScanBeanDefinitionParser#parse,

```java
public BeanDefinition parse(Element element, ParserContext parserContext) {
   //获取<context:component-scan base-package="com.jingmin.*"/>中的base-package属性值
   String basePackage = element.getAttribute(BASE_PACKAGE_ATTRIBUTE);
   //解析${xxx}占位符
   basePackage = parserContext.getReaderContext().getEnvironment().resolvePlaceholders(basePackage);
   //可能有多个位置需要扫描，根据分割符分成数组
   String[] basePackages = StringUtils.tokenizeToStringArray(basePackage,
         ConfigurableApplicationContext.CONFIG_LOCATION_DELIMITERS);

   // Actually scan for bean definitions and register them.
   ClassPathBeanDefinitionScanner scanner = configureScanner(parserContext, element);
   Set<BeanDefinitionHolder> beanDefinitions = scanner.doScan(basePackages);
   registerComponents(parserContext.getReaderContext(), beanDefinitions, element);

   return null;
}
```

`Set<BeanDefinitionHolder> beanDefinitions = scanner.doScan(basePackages);`扫描出带@Component注解的类，封装成BeanDefinition后，
最终通过`registerComponents(parserContext.getReaderContext(), beanDefinitions, element);`注册到ReaderContext，留待之后实例化。

## 参考

[Spring注解Component原理源码解析](https://www.cnblogs.com/wolf-bin/p/11667208.html)

[Spring源码解析之默认标签的解析](https://blog.csdn.net/heroqiang/article/details/78599052)

[Spring源码解析之自定义标签的解析](https://blog.csdn.net/heroqiang/article/details/78611213)

[Spring源码解析之context:component-scan标签解析](https://blog.csdn.net/heroqiang/article/details/79019359)

[Spring源码解析之@Component注解的扫描](https://blog.csdn.net/heroqiang/article/details/79019415)

[Spring中@Component，@Service等注解如何被解析？](https://blog.csdn.net/weixin_38405253/article/details/107587683)