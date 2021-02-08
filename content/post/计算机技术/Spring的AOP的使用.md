---
title: "Spring的AOP的使用"
date: 2021-02-08
draft: false
categories: 
- Spring
tags:
- Spring AOP
---

# Spring的AOP的使用

在Spring中可以进行面向切面的编程。

这里的意思是，

- Spring AOP实现了对方法织入增强
- Spring可以集成专门的AOP框架AspectJ。

一般直接考虑用Spring AOP即可。当需要对属性织入、多次织入、加载时织入、优化编织效果等再考虑集成AspectJ。

- Spring AOP官方文档（概念与使用）：https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#aop
  翻译：https://docs.huihoo.com/spring/2.0.x/zh-cn/aop.html

- Spring AOP官方文档（低级API）：https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#aop-api
  翻译：https://docs.huihoo.com/spring/2.0.x/zh-cn/aop-api.html

虽然翻译的版本是Spring2.0的老版本，但至少aop章节是没多少改动的。

如果查看Spring的源码，会发现Spring内部使用的是低级API：Advice, Advisor, PointCut等。
但如果是我们使用AOP，直接引入aspectjweawer.jar包，使用AspectJ的注解即可。（注意，虽然用的是AspectJ的注解，但是底层用是Spring AOP的低级API，这也有利于项目将来集成AspectJ）

参考下面一篇博客，从Spring AOP的低级API使用示例讲起，到最终简化为使用AspectJ形式的注解结束：https://www.huaweicloud.com/articles/f4605f69bb89c46f54ab583a3b2ae3eb.html

有一个切面执行顺序的问题：

多个切面Aspect，类上标注的@Order越小越先切入，越后退出，这个没问题。（毕竟是用代理实现的，相当于有多层代理）

问题是同一个Aspect中，@Before，@After，@AfterReturning，@AfterThrowing，@Around注解的方法可能都切入增强了某个连接点，那么连接点上这几个方法的执行顺序是？

https://blog.csdn.net/qqXHwwqwq/article/details/51678595和https://www.hicode.club/articles/2018/03/02/1550590750209.html测试的结果是依次执行doAround方法，doBefore方法。然后执行method方法，最后依次执行doAfter、doAfterReturn方法。

我测试的结果是先进入@Around增强，然后执行@Before增强，然后是被代理切入的方法，然后是@AfterReturning增强，@After增强，最后从@Around增强退出。

stackoverflow上有人说这个与Spring版本相关。