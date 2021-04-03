---
title: "SpringBoot下配置Mybatis多数据源与事务"
date: 2021-02-08
draft: true
categories: 
- Spring
- Mybatis
- 数据源
- 事务管理
tags:
- Spring
- Mybatis
- 多数据源
- 分布式事务管理
---

# 多数据源的实现方法

参考博客：https://zhuanlan.zhihu.com/p/130492982

有些场景下，应用中可能会用到多个数据源。需要添加多个数据源，可能要切换数据源，可能要在多个数据源上使用事务（保证操作的原子性、数据的一致性等等）。

参考的博文里介绍了3种实现方式：

```
1、扩展Spring的AbstractRoutingDataSource,在使用时动态切换
2、通过Mybatis 配置不同的 Mapper 使用不同的 SqlSessionTemplate
3、分库分表中间件，比如Sharding-JDBC 、Mycat等。
```

这里实践了一下第二种方式。注意：多数据源的事务需要特别注意，一不小心就会失效，或者造成数据不一致。

# SpringBoot集成MyBatis并配置多数据源

### 多数据源配置

参考：https://github.com/alibaba/druid/tree/master/druid-spring-boot-starter
参考：https://juejin.cn/post/6844903939104571400
参考：https://zhuanlan.zhihu.com/p/130492982
官网参考：http://mybatis.org/spring-boot-starter/mybatis-spring-boot-autoconfigure/
参考：https://jluncc.github.io/2019/09/22/springboot-mybatis-multidatabase/

application.properties

```properties
#多数据源配置，参考：https://github.com/alibaba/druid/tree/master/druid-spring-boot-starter
spring.datasource.druid.one.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.druid.one.url=jdbc:mysql://localhost:3306/test_amount?useUnicode=true&characterEncoding=utf-8
spring.datasource.druid.one.username=ole
spring.datasource.druid.one.password=w12345678

spring.datasource.druid.two.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.druid.two.url=jdbc:mysql://localhost:3306/test_amount2?useUnicode=true&characterEncoding=utf-8
spring.datasource.druid.two.username=ole
spring.datasource.druid.two.password=w12345678

mybatis1.type-aliases-package=com.jingmin.lockdemo.model
mybatis1.mapper-locations=classpath*:mapper/*.xml
mybatis2.type-aliases-package=com.jingmin.lockdemo.model2
mybatis2.mapper-locations=classpath*:mapper2/*.xml
```

这里用的是阿里的数据源druid，配了两个。

```java
@Configuration
@AutoConfigureBefore(DataSourceAutoConfiguration.class)
public class MultiDataSourceConfig {
    /**
     * dataSource 这个数据源交给 MybatisAutoConfiguration中默认的sqlSessionFactory使用
     */
//    @Primary
    @Bean
    @ConfigurationProperties("spring.datasource.druid.one")
    public DataSource dataSource() {
        return DruidDataSourceBuilder.create().build();
    }

    /**
     * dataSource2 这个数据源给自定义的sqlSessionFactory2使用
     */
    @Bean
    @ConfigurationProperties("spring.datasource.druid.two")
    public DataSource dataSource2() {
        return DruidDataSourceBuilder.create().build();
    }
}
```

配置Mybatis-Spring的SqlSessionTemplate，也配两个，各数据源各用各的

第一个sqlSessionTemplate：

```java
@Configuration
//下面使用数据源datasource1,配置了sqlSessionFactory和sqlSessionTemplate，
//在其中加入mapper2中接口的代理类
@MapperScan(basePackages = "com.jingmin.lockdemo.dao",
        sqlSessionTemplateRef = "sqlSessionTemplate")
public class SqlSession1Config {
    @Bean
    @ConfigurationProperties(prefix = "mybatis1")
    public MybatisProperties mybatisProperties() {
        return new MybatisProperties();
    }

    /**
     * sqlSessionFactory 第1个
     */
    @Bean
    @Primary
    public SqlSessionFactory sqlSessionFactory(@Qualifier("dataSource") DataSource dataSource,
                                               @Qualifier("mybatisProperties") MybatisProperties properties
    ) throws Exception {
        SqlSessionFactoryBean factory = new SqlSessionFactoryBean();
        factory.setDataSource(dataSource);
        factory.setVfs(SpringBootVFS.class);
        if (properties.getConfigurationProperties() != null) {
            factory.setConfigurationProperties(properties.getConfigurationProperties());
        }
        if (StringUtils.hasLength(properties.getTypeAliasesPackage())) {
            factory.setTypeAliasesPackage(properties.getTypeAliasesPackage());
        }
        if (properties.getTypeAliasesSuperType() != null) {
            factory.setTypeAliasesSuperType(properties.getTypeAliasesSuperType());
        }
        if (StringUtils.hasLength(properties.getTypeHandlersPackage())) {
            factory.setTypeHandlersPackage(properties.getTypeHandlersPackage());
        }
        if (!ObjectUtils.isEmpty(properties.resolveMapperLocations())) {
            factory.setMapperLocations(properties.resolveMapperLocations());
        }
        return factory.getObject();
    }

    /**
     * sqlSessionTemplate 第1个
     */
    @Bean
    @Primary
    public SqlSessionTemplate sqlSessionTemplate(@Qualifier("sqlSessionFactory") SqlSessionFactory sqlSessionFactory) {
        return new SqlSessionTemplate(sqlSessionFactory);
    }
}
```

sqlSessionTemplate2 ：

```java
@Configuration
//下面使用数据源datasource2,配置了sqlSessionFactory2和sqlSessionTemplate2，
//在其中加入mapper2中接口的代理类
@MapperScan(basePackages = "com.jingmin.lockdemo.dao2",
        sqlSessionTemplateRef = "sqlSessionTemplate2")
public class SqlSession2Config {
    @Bean
    @ConfigurationProperties(prefix = "mybatis2")
    public MybatisProperties mybatisProperties2() {
        return new MybatisProperties();
    }

    /**
     * sqlSessionFactory2 第二个
     */
    @Bean
    public SqlSessionFactory sqlSessionFactory2(@Qualifier("dataSource2") DataSource dataSource,
                                                @Qualifier("mybatisProperties2") MybatisProperties properties
    ) throws Exception {
        SqlSessionFactoryBean factory = new SqlSessionFactoryBean();
        factory.setDataSource(dataSource);
        factory.setVfs(SpringBootVFS.class);
        if (properties.getConfigurationProperties() != null) {
            factory.setConfigurationProperties(properties.getConfigurationProperties());
        }
        if (StringUtils.hasLength(properties.getTypeAliasesPackage())) {
            factory.setTypeAliasesPackage(properties.getTypeAliasesPackage());
        }
        if (properties.getTypeAliasesSuperType() != null) {
            factory.setTypeAliasesSuperType(properties.getTypeAliasesSuperType());
        }
        if (StringUtils.hasLength(properties.getTypeHandlersPackage())) {
            factory.setTypeHandlersPackage(properties.getTypeHandlersPackage());
        }
        if (!ObjectUtils.isEmpty(properties.resolveMapperLocations())) {
            factory.setMapperLocations(properties.resolveMapperLocations());
        }
        return factory.getObject();
    }

    /**
     * sqlSessionTemplate2 第二个
     */
    @Bean
    public SqlSessionTemplate sqlSessionTemplate2(@Qualifier("sqlSessionFactory2") SqlSessionFactory sqlSessionFactory) {
        return new SqlSessionTemplate(sqlSessionFactory);
    }
}
```

那么，如果要dataSource对应的数据库，就使用sqlSessionTemplate；如果要dataSource2对应的数据库，就使用sqlSessionTemplate2.

使用多数据源，默认的事务处理会出现问题。

一方面，SpringBoot默认不会为多数据源配置事务管理器：

```java
@Configuration(proxyBeanMethods = false)
@ConditionalOnClass({ JdbcTemplate.class, TransactionManager.class })
@AutoConfigureOrder(Ordered.LOWEST_PRECEDENCE)
@EnableConfigurationProperties(DataSourceProperties.class)
public class DataSourceTransactionManagerAutoConfiguration {

	@Configuration(proxyBeanMethods = false)
    //注意这里，只为单数据源的情况进行配置。
	@ConditionalOnSingleCandidate(DataSource.class)
	static class JdbcTransactionManagerConfiguration {

		@Bean
		@ConditionalOnMissingBean(TransactionManager.class)
		DataSourceTransactionManager transactionManager(Environment environment, DataSource dataSource,
				ObjectProvider<TransactionManagerCustomizers> transactionManagerCustomizers) {
			DataSourceTransactionManager transactionManager = createTransactionManager(environment, dataSource);
			transactionManagerCustomizers.ifAvailable((customizers) -> customizers.customize(transactionManager));
			return transactionManager;
		}

		private DataSourceTransactionManager createTransactionManager(Environment environment, DataSource dataSource) {
			return environment.getProperty("spring.dao.exceptiontranslation.enabled", Boolean.class, Boolean.TRUE)
					? new JdbcTransactionManager(dataSource) : new DataSourceTransactionManager(dataSource);
		}
	}
}
```

另一方面，Spring不为多数据源进行默认配置也是有原因的。因为原来的事务提交方式（一阶段提交）无法保证多数据源下的数据一致性。
假如，分别给数据源1和数据源2按原来的方式，各配置一个事务管理器。比如说，某个操作需要保证原子性，而这个操作内部又可以分为两步，第一步使用了数据源1，第二步使用了数据源2。
如果第一步中出现了错误，第一步使用的数据源1就会进行回滚，数据2还没有使用，这个时候是没有数据一致性问题的。
但是当第一步执行完成并提交数据源1后，第二步执行出错回滚数据源，这个时候数据源1并不会回滚。！！！

### 多数据源事务问题

参考：https://v2ex.com/t/611462
参考：https://stackoverflow.com/questions/48954763/spring-transactional-with-a-transaction-across-multiple-data-sources
参考：https://blog.csdn.net/weixin_41715077/article/details/83105033

v2ex论坛上https://v2ex.com/t/611462也谈到了多数据源事务问题

> **[hantsy](https://v2ex.com/member/hantsy)**  2019-10-22 11:13:16 +08:00

stackoverflow上https://stackoverflow.com/questions/48954763/spring-transactional-with-a-transaction-across-multiple-data-sources也讨论了这个问题，建议使用Spring中的ChainedTransactionManager，当然现在它现在迁移到了Spring Data Commons中。



之前也说到了SpringBoot中使用多数据源，并不会为我们自动配置事务管理器。就算我们手动为各数据源配置了事务管理器，也会涉及到一个原子操作中，事务管理器的切换问题。也有人称为分布式事务问题。

解决这样的问题，一般有两个思路：
1.两阶段提交
2.尽最大可能一阶段提交

Spring提供了ChainedTransactionManager，它将多个事务串起来执行，最后一起commit。当然，最后多个事务依次commit的过程也可能会失败，已commit的已经无法回滚，还未commit的将全部回滚，所以并不能保证数据的一致性。它只是尽最大可能一阶段提交。

```java
/**
     * 这个才是我们最后要用到的事务管理器,使用链式事务管理器进行管理各个数据源的事务。
     * 如果出现跨数据源中操作，所有操作都进行完后，才统一进行事务的提交。
     * 如果操作过程中出错，所有操作都可以回滚。
     * 但是要注意：统一提交的过程中，仍有可能出错，这时已经提交的事务就无法回滚了，只能回滚还未提交的。
     * 所以，链式事务管理，并不能解决分布式事务的数据一致性问题，只是尽最大可能一次提交。
     */
@Bean
public PlatformTransactionManager transactionManager(
    @Qualifier("dataSource") DataSource dataSource,
    @Qualifier("dataSource2") DataSource dataSource2) {
    PlatformTransactionManager transactionManager1 = new DataSourceTransactionManager(dataSource);
    PlatformTransactionManager transactionManager2 = new DataSourceTransactionManager(dataSource2);
    return new ChainedTransactionManager(transactionManager1, transactionManager2);
}
```

举个例子，客户下单，员工可以拿提成（加薪）。订单操作在订单库中进行，员工加薪操作在员工信息库中进行。
使用上面的链式事务管理器做如下测试

```
/**
* 顾客下单，员工加薪1元
* 这里使用了多数据源，分别给配了一个事务管理器，然后使用链式事务管理器综合管理
*/
@Transactional(rollbackFor = Exception.class)
public void safeOrderAndEmpAddSalary(Long custId, List<Product> productList, Long empId) throws Exception {
//这里是订单数据库进行操作：顾客下单
orderSecureWithMysqlPessimisticLock(custId, productList);
//测试点1
//if (false) throw new Exception("测试在这个步骤出错,能否保证数据一致性，即前面的事务能否回滚");
//这里是员工信息数据库进行操作：员工加薪
empDao.addSalary(empId, new BigDecimal(1));
//测试点2
//if (true) throw new Exception("测试在这个步骤出错,能否保证数据一致性，即前面的事务能否回滚");
}
```

分别在上面两个测试点扔出Exception，发现两个数据库都可以做到回滚，这是我们期待看到的。一般而言，做到这种程度的一致性就可以了。
但是整个safeOrderAndEmpAddSalary方法完成，最后进行事务统一提交时，仍有可能出错：比如订单库完成提交，但是员工信息库突然down机了，这个时候，订单库是无法进行回滚的。
这种情况，如果出现几率不大，影响也不严重（老板可能这样觉得），是可以不做处理的。
如果必须要处理，只需要前面的操作（顾客下单时，多做一次查询判断，防止重复提交即可）。所以才叫尽最大可能一次提交。

源码：https://github.com/ole12138/lock-demo