---
title: "转载：Maven pom.xml中的元素modules、parent、properties以及import"
date: 2021-01-06T10:00:39+08:00
draft: false
categories: 
- 工具
series:
- Maven
tags:
- Maven
- 依赖管理
- Java构建工具
---

# Maven pom.xml中的元素modules、parent、properties以及import



非原创，转载来源：https://www.cnblogs.com/youzhibing/p/5427130.html

另，Maven文档链接：https://maven.apache.org/guides/introduction/introduction-to-the-pom.html

maven的核心是pom.xml。印象最深的就是如下四个元素：modules、parent、properties、import。

## modules

从字面意思来说，module就是模块，而pom.xml中的modules也正是这个意思，用来管理同个项目中的各个模块；如果maven用的比较简单，或者说项目的模块在pom.xml没进行划分，那么此元素是用不到的；不过一般大一点的项目是要用到的。

### 需求场景

如果我们的项目分成了好几个模块，那么我们构建的时候是不是有几个模块就需要构建几次了（到每个模块的目录下执行mvn命令）？当然，你逐个构建没问题，但是非要这么麻烦的一个一个的构建吗，那么简单的做法就是使用聚合，一次构建全部模块。

### 具体实现

a.既然使用聚合，那么就需要一个聚合的载体，先创建一个普通的maven项目account-aggregator,如下图：

![img](data:image/png;base64,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)

因为是个聚合体，仅仅负责聚合其他模块，那么就只需要上述目录，该删除的就删了；注意的是pom文件的书写：

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.youzhibing.account</groupId>
  <artifactId>account-aggregator</artifactId>
  <version>1.0.0-SNAPSHOT</version>
  <packaging>pom</packaging>

  <name>Account Aggrregator</name>
  <url>http://maven.apache.org</url>
  
  <modules>　　　 <!-- 模块都写在此处 -->
      <module>account-register</module>
      <module>account-persist</module>
  </modules>

</project>
```

b.创建子模account-register、account-persist:右击account-aggregator，new --> other --> Maven，选择Maven Module，创建maven模块。

![img](https://picgo12138.oss-cn-hangzhou.aliyuncs.com/md/2021年1月6日下载1.png)

c.创建完成后，项目结构如下，那么此时account-aggregator可以收缩起来了，我们操作具体子模块就好了。

![2021年1月6日下载2](https://picgo12138.oss-cn-hangzhou.aliyuncs.com/md/2021年1月6日下载2.png)

d.注意点，当我们打开包结构的子模块的pom文件时，发现离预期的多了一些内容，我们坐下处理就好了

![2021年1月6日下载3](https://picgo12138.oss-cn-hangzhou.aliyuncs.com/md/2021年1月6日下载3.png)

e.那么编码完了之后，我们只需要构建account-aggregator就好了，所有的子模块都会构建。

## parent

继承，和java中的继承相当，作用就是复用

### 需求场景

若每个子模块都都用的了spring，那么我们是不是每个子模块都需要单独配置spring依赖了?,这么做是可以的，但是我们有更优的做法，那就是继承，用parent来实现。

### 具体实现

a.配置父pom.xml

我就用聚合pom来做父pom,配置子模块的公共依赖。

父(account-aggregator)pom.xml :

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.youzhibing.account</groupId>
  <artifactId>account-aggregator</artifactId>
  <version>1.0.0-SNAPSHOT</version>
  <packaging>pom</packaging>

  <name>Account Aggrregator</name>
  <url>http://maven.apache.org</url>
  
  <modules>
      <!-- 模块都写在此处 -->
      <module>account-register</module>
      <module>account-persist</module>
  </modules>
  
  <dependencies> <!-- 配置共有依赖 -->
      <!-- spring 依赖 -->
      <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-core</artifactId>
        <version>4.0.2.RELEASE</version>
    </dependency>
      <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-beans</artifactId>
        <version>4.0.2.RELEASE</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>4.0.2.RELEASE</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context-support</artifactId>
        <version>4.0.2.RELEASE</version>
    </dependency>
    
      <!-- junit 依赖 -->
      <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.7</version>
      <scope>test</scope>
    </dependency>
  </dependencies>

</project>
```

b.account-register的pom.xml :

```xml
<?xml version="1.0"?>
<project xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd" xmlns="http://maven.apache.org/POM/4.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <modelVersion>4.0.0</modelVersion>
  <parent>
    <groupId>com.youzhibing.account</groupId>
    <artifactId>account-aggregator</artifactId>
    <version>1.0.0-SNAPSHOT</version>
    <relativePath>../pom.xml</relativePath> <!-- 与不配置一样，默认就是寻找上级目录下得pom.xml -->
  </parent>
  <artifactId>account-register</artifactId>
  <name>account-register</name>
  <url>http://maven.apache.org</url>
  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
  </properties>
  <dependencies>    <!-- 配置自己独有依赖 -->
      <dependency>
          <groupId>javax.mail</groupId>
          <artifactId>mail</artifactId>
          <version>1.4.3</version>
      </dependency>
    <dependency>
      <groupId>com.icegreen</groupId>
      <artifactId>greenmail</artifactId>
      <version>1.4.1</version>
      <scope>test</scope>
    </dependency>
  </dependencies>
</project>
```

c.account-persist的pom.xml :

```xml
<?xml version="1.0"?>
<project xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd" xmlns="http://maven.apache.org/POM/4.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <modelVersion>4.0.0</modelVersion>
  <parent>
    <groupId>com.youzhibing.account</groupId>
    <artifactId>account-aggregator</artifactId>
    <version>1.0.0-SNAPSHOT</version>
  </parent>
  <artifactId>account-persist</artifactId>
  <name>account-persist</name>
  <url>http://maven.apache.org</url>
  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
  </properties>
  <dependencies>    <!-- 配置自己独有依赖 -->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-jdbc</artifactId>
        <version>4.0.2.RELEASE</version>
    </dependency>
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>druid</artifactId>
        <version>1.0.16</version>
    </dependency>
  </dependencies>
</project>
```

d.依赖的jar包全部ok，需要做的则是在各个模块中进行代码开发了！

### 依赖管理

继承可以消除重复，那是不是就没有问题了？ 答案是存在问题，假设将来需要添加一个新的子模块account-util，该模块只是提供一些简单的帮助工具，不需要依赖spring、junit，那么继承后就依赖上了，有没有什么办法了？ 有，maven已经替我们想到了，那就是dependencyManagement元素，既能让子模块继承到父模块的依赖配置，又能保证子模块依赖使用的灵活性。在dependencyManagement元素下得依赖声明不会引入实际的依赖，不过它能够约束dependencies下的依赖使用。

在父pom.xml中配置dependencyManagement元素:

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.youzhibing.account</groupId>
  <artifactId>account-aggregator</artifactId>
  <version>1.0.0-SNAPSHOT</version>
  <packaging>pom</packaging>

  <name>Account Aggrregator</name>
  <url>http://maven.apache.org</url>
  
  <modules>
      <!-- 模块都写在此处 -->
      <module>account-register</module>
      <module>account-persist</module>
  </modules>
  
  <dependencyManagement>
      <dependencies> <!-- 配置共有依赖 -->
      <!-- spring 依赖 -->
      <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-core</artifactId>
        <version>4.0.2.RELEASE</version>
    </dependency>
      <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-beans</artifactId>
        <version>4.0.2.RELEASE</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>4.0.2.RELEASE</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context-support</artifactId>
        <version>4.0.2.RELEASE</version>
    </dependency>
    
      <!-- junit 依赖 -->
      <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.7</version>
      <scope>test</scope>
    </dependency>
  </dependencies>
  </dependencyManagement>
</project>
```

account-persist的pom.xml(account-register也一样) ：

```xml
<?xml version="1.0"?>
<project xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd" xmlns="http://maven.apache.org/POM/4.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <modelVersion>4.0.0</modelVersion>
  <parent>
    <groupId>com.youzhibing.account</groupId>
    <artifactId>account-aggregator</artifactId>
    <version>1.0.0-SNAPSHOT</version>
  </parent>
  <artifactId>account-persist</artifactId>
  <name>account-persist</name>
  <url>http://maven.apache.org</url>
  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
  </properties>
  <dependencies>
      <!-- spring 依赖 -->
      <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-core</artifactId>
    </dependency>
      <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-beans</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context-support</artifactId>
    </dependency>
    
      <!-- junit 依赖 -->
      <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-jdbc</artifactId>
        <version>4.0.2.RELEASE</version>
    </dependency>
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>druid</artifactId>
        <version>1.0.16</version>
    </dependency>
  </dependencies>
</project>
```

使用这种依赖管理机制似乎不能减少太多的POM配置，就少了version(junit还少了个scope)，感觉没啥作用呀；其实作用还是挺大的，父POM使用dependencyManagement能够统一项目范围中依赖的版本，当依赖版本在父POM中声明后，子模块在使用依赖的时候就无须声明版本，也就不会发生多个子模块使用版本不一致的情况，帮助降低依赖冲突的几率。如果子模块不声明依赖的使用，即使该依赖在父POM中的dependencyManagement中声明了，也不会产生任何效果。

## import

import只在dependencyManagement元素下才有效果，作用是将目标POM中的dependencyManagement配置导入并合并到当前POM的dependencyManagement元素中，如下就是讲account-aggregator中的dependencyManagement配置导入并合并到当前POM中。

```xml
<dependencyManagement>
      <dependencies>
        <dependency>
            <groupId>com.youzhibing.account</groupId>
              <artifactId>account-aggregator</artifactId>
              <version>1.0.0-SNAPSHOT</version>
            <type>pom</type>
              <scope>import</scope>
        </dependency>
      </dependencies>
  </dependencyManagement>
```

## properties

通过`<properties>`元素用户可以自定义一个或多个Maven属性，然后在POM的其他地方使用${属性名}的方式引用该属性，这种做法的最大意义在于消除重复和统一管理。

Maven总共有6类属性，内置属性、POM属性、自定义属性、Settings属性、java系统属性和环境变量属性；

### 内置属性

两个常用内置属性 `${basedir}` 表示项目跟目录，即包含pom.xml文件的目录；`${version}` 表示项目版本

### pom属性

用户可以使用该类属性引用POM文件中对应元素的值。如`${project.artifactId}`就对应了`<project> <artifactId>`元素的值，常用的POM属性包括：

```
${project.build.sourceDirectory}:项目的主源码目录，默认为src/main/java/

${project.build.testSourceDirectory}:项目的测试源码目录，默认为src/test/java/

${project.build.directory} ： 项目构建输出目录，默认为target/

${project.outputDirectory} : 项目主代码编译输出目录，默认为target/classes/

${project.testOutputDirectory}：项目测试主代码输出目录，默认为target/testclasses/

${project.groupId}：项目的groupId

${project.artifactId}：项目的artifactId

${project.version}：项目的version,与${version} 等价

${project.build.finalName}：项目打包输出文件的名称，默认为${project.artifactId}-${project.version}
```

### 自定义属性

如下account-aggregator的pom.xml，那么继承了此pom.xml的子模块也可以用此自定义属性.

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.youzhibing.account</groupId>
  <artifactId>account-aggregator</artifactId>
  <version>1.0.0-SNAPSHOT</version>
  <packaging>pom</packaging>

  <name>Account Aggrregator</name>
  <url>http://maven.apache.org</url>
  
  <modules>
      <!-- 模块都写在此处 -->
      <module>account-register</module>
      <module>account-persist</module>
      <module>account-another</module>
  </modules>
  
  <properties>
      <!-- 定义 spring版本号 -->
    <spring.version>4.0.2.RELEASE</spring.version>
    <junit.version>4.7</junit.version>
  </properties>
  
  <dependencyManagement>
      <dependencies> <!-- 配置共有依赖 -->
      <!-- spring 依赖 -->
      <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-core</artifactId>
        <version>${spring.version}</version>
    </dependency>
      <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-beans</artifactId>
        <version>${spring.version}</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>${spring.version}</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context-support</artifactId>
        <version>${spring.version}</version>
    </dependency>
    
      <!-- junit 依赖 -->
      <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>${junit.version}</version>
      <scope>test</scope>
    </dependency>
  </dependencies>
  </dependencyManagement>
</project>
```

### settings属性

与POM属性同理，用户使用以settings. 开头的属性引用settings.xml文件中的XML元素的值。

### Java系统属性

所有java系统属性都可以用Maven属性引用，如`${user.home}`指向了用户目录。

### 环境变量属性

所有环境变量属性都可以使用以env. 开头的Maven属性引用，如`${env.JAVA_HOME}`指代了JAVA_HOME环境变量的的值。

## 聚合与继承的关系

1.聚合主要是为了方便快速构建项目，继承主要是为了消除重复配置；

2.对于聚合模块而言，它知道有哪些被聚合的模块，但那些被聚合的模块不知道这个聚合模块的存在；对于继承的父pom而言，它不知道有哪些子模块继承它，但那些子模块都必须知道自己的父POM是什么；

3.聚合POM与继承中的父POM的packaging都必须是pom；同时，聚合模块与继承中的父模块除了POM外，都没有实际的内容

## 结束语

maven越来越流行，这方面的资料也是越来越多，《Maven实战》给我的感觉就相当不错，本博客的内容大多取自其中；网上资料也越来越多，就博客园中就有不少；

最后强调一点：**看了是好，实践更好，写博客记录下来那是最好！**