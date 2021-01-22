---
title: "Maven打包"
date: 2021-01-22T13:47:18+08:00
draft: false
categories: 
- 工具
series:
- Maven
tags:
- Maven
- jar包
- Java构建工具
---

# Maven打jar包

实际开发的项目一般都要打成jar包或者war包。

如果是独立的java程序，打成jar包后，在安装了java运行时的环境中，命令行即可直接运行：

```
java -jar xxx.jar
```

如果是传统的java web项目，打成war包后，放入tomcat或其他java web容器中，即可直接运行。

如果使用springboot开发java web项目，由于springboot默认集成javaweb容器，也可以打成jar包，在命令行直接运行。

## 独立java程序打jar包

首先，项目的pom文件要添加一行，设置打包格式为jar格式：

```xml
<packaging>jar</packaging>
```

然后要在项目的pom.xml文件中添加`maven-resources-plugin`,用于将项目的配置文件复制到将要生成的jar包外，方便修改配置。（这里将项目的配置文件复制到了输出文件夹的/resources目录下）。

```xml
<!-- Maven插件的汇总介绍： https://maven.apache.org/plugins/index.html -->

<!-- 资源/配置文件的复制插件 -->
<!-- https://maven.apache.org/plugins/maven-resources-plugin/ -->
<!-- 默认：src/main/resources下的资源和pom.xml下project.build.resources标签下的资源都会被自动复制到源码编译后的输出目录-->
<!-- 默认：测试源码目录的资源会被自动输出到测试输出目录-->
<!-- 如果想要把配置/资源输出到指定目录，需要作自定义配置-->
<plugin>
    <artifactId>maven-resources-plugin</artifactId>
    <version>3.2.0</version>
    <!-- 自定义配置：该过程的作用是用于复制指定的资源文件到指定位置-->
    <!--可用于将配置文件放到jar包外面，方便修改 -->
    <executions>
        <execution>
            <id>copy-resources</id>
            <phase>package</phase>
            <goals>
                <goal>copy-resources</goal>
            </goals>
            <configuration>
                <resources>
                    <resource>
                        <directory>src/main/resources</directory>
                        <includes>
                            <include>*.properties</include>
                            <include>log4j2.xml</include>
                        </includes>
                    </resource>
                </resources>
                <outputDirectory>${project.build.directory}/resources/</outputDirectory>
            </configuration>
        </execution>
    </executions>
</plugin>
```

然后要在项目的pom.xml文件中添加`maven-dependency-plugin`,用于将项目的依赖复制到将要生成的jar包外。（这里将项目的依赖复制到了输出文件夹的/lib目录下）。

```xml
<!--依赖复制与分析插件-->
<!--https://maven.apache.org/plugins/maven-dependency-plugin/-->
<!-- 这里该插件的作用是用于复制依赖的jar包到指定的文件夹里 -->
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-dependency-plugin</artifactId>
    <version>3.1.2</version>
    <executions>
        <execution>
            <id>copy-dependencies</id>
            <phase>package</phase>
            <goals>
                <goal>copy-dependencies</goal>
            </goals>
            <configuration>
                <outputDirectory>${project.build.directory}/lib/</outputDirectory>
                <excludeTransitive>false</excludeTransitive>
                <!--<stripVersion>true</stripVersion>-->
                <stripVersion>false</stripVersion>
                <overWriteReleases>false</overWriteReleases>
                <overWriteSnapshots>false</overWriteSnapshots>
                <overWriteIfNewer>true</overWriteIfNewer>
            </configuration>
        </execution>
    </executions>
</plugin>
```

再者就是在项目的pom.xml文件中添加`maven-jar-plugin`,用于将项目打包成jar文件，这里排除了项目的配置文件，并修改生成jar的META-INF/MANEFEST.MF描述文件。（指引jar使用项目外的lib/目录下的依赖和项目外resources/目录下的配置文件）。

```xml
<!-- 打jar包插件 -->
<!--https://maven.apache.org/plugins/maven-jar-plugin/-->
<!-- 打jar包，以下的配置是忽略properties文件，并在MANIFEST文件中指明依赖包所在路径和配置文件所在路径 -->
<plugin>
    <artifactId>maven-jar-plugin</artifactId>
    <version>3.0.2</version>
    <configuration>
        <excludes>
            <exclude>*.properties</exclude>
            <exclude>log4j2.xml</exclude>
        </excludes>
        <archive>
            <manifest>
                <mainClass>com.jingmin.Main</mainClass>
                <addClasspath>true</addClasspath>
                <classpathPrefix>lib/</classpathPrefix>
            </manifest>
            <manifestEntries>
                <!--MANIFEST.MF 中 Class-Path 加入资源文件目录 -->
                <Class-Path>./resources/</Class-Path>
            </manifestEntries>
        </archive>
    </configuration>
</plugin>
```

在项目文件夹下或IDE中执行maven命令：

```
mvn package
```

就会在项目的输出目录下多生成2个目录和1个jar包：

```
lib/						#项目所有的依赖都自动复制到了这个目录下
resources/					#项目配置文件自动复制到了这个目录下
xxxx.jar					#生成的可执行jar包
```

然后只需要将这2个目录和1个jar包，复制到需要部署的环境中，命令行执行jar命令即可：

```
java -jar xxxx.jar
```

## java web项目打包成war

项目的pom文件添加一行，设置打包格式为war格式即可：

```xml
<packaging>war</packaging>
```

在项目文件夹下或IDE中执行maven命令：

```
mvn package
```

就会在项目的输出目录下1个war包，复制到需要部署的web容器中（比如tomcat中），项目就会自动解也运行。

## java web项目打包成jar

实际上，这是借助springboot完成的。springboot项目的pom中，自动添加了web容器的依赖，打包成的jar中自动附带了web容器，所以不需要放到web容器中运行。

通过[Spring Initializr](https://start.spring.io/)生成的Springboot项目，pom.xml文件中自带了springboot插件。如果是web项目，打包时，自动附带了web容器，生成了可在命令行直接运行的jar格式的web项目。

将jar包复制到需要部署的环境中，命令行执行jar命令，web项目就运行起来了。

```
java -jar xxxx.jar
```

