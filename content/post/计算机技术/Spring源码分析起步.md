---
title: "Spring源码分析起步"
date: 2021-01-21T12:01:24+08:00
draft: false
categories: 
- Spring
tags:
- Spring源码
---

# Spring源码分析起步

这是从网上找的一张Spring的类继承图，IDE中翻了一下源码，当前版本并没有什么变动。

![下载2021年1月22日-1](https://picgo12138.oss-cn-hangzhou.aliyuncs.com/md/下载2021年1月22日-1.png)

## 普通应用程序启动Spring前的准备

对于Spring的普通应用程序（从XML加载配置），一般是从这样的代码开始的：

```java
ApplicationContext applicationContext = new ClassPathXmlApplicationContext("applicationContext.xml");
        Message message = (Message) applicationContext.getBean("message");
```

其中`ClassPathXmlApplicationContext`的初始化过程：

```java
public ClassPathXmlApplicationContext(
      String[] configLocations, boolean refresh, @Nullable ApplicationContext parent)
      throws BeansException {
   //调用父类初始化过程和记录配置文件位置
   ...
   if (refresh) {
      //启动Spring的核心过程
      refresh();
   }
}
```

注意到`refresh()`方法是在其祖先类`AbstractApplicationContext`中定义的。

下面我们先不往下跟`refresh()`方法，先看一下web项目，就会发现web项目在启动Spring的过程中，最终同样到达了`AbstractApplicationContext`中`refresh()`方法。

## web项目启动Spring前的准备

而对于Spring的web项目（从XML加载配置），一般是在项目的web.xml添加了Spring的Listener：

```xml
<!--加载spring-->
<context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>classpath*:applicationContext.xml</param-value>
</context-param>
<listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>
```

而其中这个Listener的类实现了ServletContextListener接口

```java
public class ContextLoaderListener extends ContextLoader implements ServletContextListener {
    @Override
	public void contextInitialized(ServletContextEvent event) {
		initWebApplicationContext(event.getServletContext());
	}
    ...
}
```

根据Servlet的规范，在web项目在Context初始化的过程中javax.servlet.ServletContextListener的`contextInitialized(ServletContextEvent event)`方法会被调用。

这个方法是父类`ContextLoader`的方法

```java
public class ContextLoader {	
    public WebApplicationContext initWebApplicationContext(ServletContext servletContext){
        //日志记录以及利用servletContext配置wab应用上下文
        ...
            //继续调用本类中的方法
            configureAndRefreshWebApplicationContext(cwac, servletContext);
        //日志记录、catch Exceptions
        ...
    }
    	protected void configureAndRefreshWebApplicationContext(ConfigurableWebApplicationContext wac, ServletContext sc) {
		//继续初始化web应用环境
        ...
        //从这里启动Spring
		wac.refresh();
	}
}
```

而`ConfigurableWebApplicationContext`是`AbstractApplicationContext`的子孙类，实例`wac`的方法`refresh()`在`AbstractApplicationContext`中定义。

综上，不管是普通Spring项目，还是包含了Spring的web项目，，最终都是通过`AbstractApplicationContext`中`refresh()`方法来完成启动的。

## `AbstractApplicationContext`中`refresh()`方法

不管是普通Spring项目，还是包含了Spring的web项目，，最终都是通过`AbstractApplicationContext`中`refresh()`方法来完成启动的。

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

以上注释参考https://blog.csdn.net/qq924862077/article/details/60779831



之后若关注Spring的某一块功能，都可以在这里打打断点，查看运行过程。



## 关于Spring的进一步分析

[Spring源码分析-归田](https://juejin.cn/post/6844903593879797768)

[Spring源码分析-陈彦斌](https://www.cnblogs.com/chenyanbin/p/11756034.html)

[Spring 加载 *.properties 文件的源码分析](https://blog.csdn.net/dalinsi/article/details/53037957)