title: Java9新特性系列（module&maven&starter）
date: 2018-02-11 22:30:38
categories: Java
tags: [Java,Java9新特性]
---
上篇已经深入分析了[Java9中的模块化](http://hellomypastor.net/2018/02/10/Java9%E6%96%B0%E7%89%B9%E6%80%A7%E7%B3%BB%E5%88%97%EF%BC%88%E6%B7%B1%E5%85%A5%E7%90%86%E8%A7%A3%E6%A8%A1%E5%9D%97%E5%8C%96%EF%BC%89/)，有读者又提到了module与starter是什么关系？本篇将进行分析。

>首先先回顾下module与maven/gradle的关系：


# module与maven/gradle是什么关系？

模块化的依赖关系，很容易让人联想到mven和gradle，这个在上篇中也提及，后来有读者也提出module和maven是什么关系？解答如下：

Maven 有两个主要的特征：依赖管理和构建管理。
依赖管理即可以决定库中的版本并从仓库中下载下来。
构建管理即管理开发过程中的任务，包括初始化、编译、测试、打包等。

Module是系统内置用于表述组件之间的关系，对于版本的管理还是处于最原始的状体，管理一种强制的依赖关系。

总结一下：Maven还是主要负责构建管理，Modules 对于像Maven这样的构建工具（build tools）来说扮演的是辅助补充的角色。因为这些构建工具在构建时建立的依赖关系图谱和他们版本可以根据Module来创建，Module强制确定了module和artifacts之间的依赖关系，而Maven对于依赖并非是强制的。
具体可参考StackOverflow上的一篇问答：[Project Jigsaw vs Maven on StackOverflow](https://stackoverflow.com/questions/39844602/project-jigsaw-vs-maven)

<!--more-->

>说到starter，如果用过SpringBoot的开发者可能很熟悉，Starter主要用来简化依赖用的。


# Starter是什么？
**Starter主要用来简化依赖用的**
+ Starter前时代

在Starter之前做MVC时要引入日志组件，比如log4j，那么需要找到依赖的log4j的版本，有的时候版本不匹配也会出现问题，因为我们也不知道依赖什么版本，那么Starter就应运而生。

+ Starter

我们要在SpringBoot中引入WebMVC的支持时，我们只需引入这个模块spring-boot-starter-web，有了Starter之后，log4j就自动引入了，也不用关心版本这些问题，将这个模块如果解压包出来会发现里面什么都没有，只定义了一些POM依赖，所以说Starter的作用主要是简化依赖。

+ SpringBoot Starters

`《Spring Boot in Action》`的附录B列出了SpringBoot Starters的内容：

| Starter(Group ID: org.springframework.boot) | 传递依赖 | 
| ------------- |-------------|
| spring-boot-starter-web | org.springframework.boot:spring-boot-starter<br>org.springframework.boot:spring-boot-starter-tomcat<br>org.springframework.boot:spring-boot-starter-validation<br>com.fasterxml.jackson.core:jackson-databind<br>org.springframework:spring-web<br>org.springframework:spring-webmvc |
| spring-boot-starter-log4j2 |org.apache.logging.log4j:log4j-slf4j-impl<br>org.apache.logging.log4j:log4j-api<br>org.apache.logging.log4j:log4j-core<br>org.slf4j:jcl-over-slf4j<br>org.slf4j:jul-to-slf4j |
|...|....|

# 如何定义自己的Starter？
+ 基于已有starter进行扩展（pom.xml）
```xml
<!-- 依赖管理 -->
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-dependencies</artifactId>
            <version>1.5.2.RELEASE</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
<!-- 添加依赖 -->
<dependencies>
    <!-- 自动配置依赖 -->
    <dependency>  
        <groupId>org.springframework.boot</groupId>  
        <artifactId>spring-boot-autoconfigure</artifactId> 
        <version>1.5.2.RELEASE</version>  
    </dependency>  
    <dependency>  
        <groupId>org.springframework.boot</groupId>  
        <artifactId>spring-boot-configuration-processor</artifactId>  
        <version>1.5.2.RELEASE</version>  
        <optional>true</optional>  
    </dependency> 
</dependencies>
```
+ 定义自动配置配置类
```java
package net.hellomypastor.springboot.starter;

import org.springframework.boot.context.properties.ConfigurationProperties;

@ConfigurationProperties(prefix="dog.proterties")
public class DogProperties {
    private String name;
    private int age;
    //set、get
}
```

```java
package net.hellomypastor.springboot.starter;

public class DogService {
    private DogProperties dogProperties;

    public DogService() {

    }

    public DogService(DogProperties dogProperties) {
        this.dogProperties = dogProperties;
    }

    public String getDogName() {
        return dogProperties.getName();
    }

    public int getDogAge() {
        return dogProperties.getAge();
    }
}
```

```java
package net.hellomypastor.springboot.starter;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.autoconfigure.condition.ConditionalOnClass;
import org.springframework.boot.autoconfigure.condition.ConditionalOnMissingBean;
import org.springframework.boot.autoconfigure.condition.ConditionalOnProperty;
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
@EnableConfigurationProperties(DogProperties.class)
@ConditionalOnClass(DogService.class)
@ConditionalOnProperty(prefix="dog.proterties", value="enabled", matchIfMissing=true)
public class DogAutoConfiguration {
    @Autowired
    private DogProperties dogProperties;

    @Bean
    @ConditionalOnMissingBean(DogService.class)
    public DogService personService() {
        DogService dogService = new DogService(dogProperties);
        return dogService;
    }
}

```
+ 修改META-INF/spring.factories
```properties
org.springframework.boot.autoconfigure.EnableAutoConfiguration=net.hellomypastor.configure.DogAutoConfiguration
```
+ 打包
>spring官方说明定义starter时最好是按`*-spring-boot-starter`的格式进行命名
```shell
mvn clean install
```
+ 使用

将上述starter作为依赖，并配置application.properties：
```properties
dog.properties.name=dog2018
dog.properties.age=1
```
那么在类中只要注入DogService就能获取到配置的值：
```java
package net.hellomypastor.springboot.starter;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/dog")
public class DogController {
    @Autowired
    private DogService dogService;

    @GetMapping("name")
    public String getDogName() {
        return dogService.getDogName();
    }

    @GetMapping("age")
    public int getDogAge() {
        return dogService.getDogAge();
    }
}
```
测试请求结果：
```xml
http:127.0.0.1:8080/starterdemo/dog/name
结果：dog2018
http:127.0.0.1:8080/starterdemo/dog/age
结果：1
```

>好了，本期就讲到这里，下期我们讲讲Java9中到另一个新工具`JShell`，敬请期待～