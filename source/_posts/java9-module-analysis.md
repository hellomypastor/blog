title: Java9新特性系列（深入理解模块化）
date: 2018-02-10 22:28:43
categories: Java
tags: [Java,Java9新特性]
---
前两篇文章介绍了Java9新特性系列`JDK与JRE`以及`模块化系统: Jigsaw->Modularity`，本篇我们将深入理解模块化。
# 模块化如何体现？
如下图所示，Jdk8与Jdk9的目录结构，这个在之前的jdk与jre的文章已经提及。
![](https://user-gold-cdn.xitu.io/2018/2/10/1617f5c7e2af83ec?w=807&h=560&f=png&s=19820)

![](https://user-gold-cdn.xitu.io/2018/2/10/1617f5c7e292d073?w=1240&h=445&f=png&s=44939)

从上面两张图对比可以发现：
JDK8:
在Jdk8中有两个重要的jar，即rt.jar与tools.jar：
![](https://user-gold-cdn.xitu.io/2018/2/10/1617f5c7e2b74006?w=988&h=36&f=png&s=13928)

![](https://user-gold-cdn.xitu.io/2018/2/10/1617f5c7e29267ed?w=1020&h=30&f=png&s=15027)

在Jdk8中有jre，在jre/lib目录中有一个rt.jar（大小约64M），即rutime，提供了运行环境所用到的一些类库；在lib目录有一个tools.jar（大小约17M），是java中最基本的包，里面包含了从java中最重要的lang包到各种高级功能如可视化swing的包。

JDK9：
JDK9中没有jre，没有rt.jar，没有tools.jar，都是一个一个模块
![](https://user-gold-cdn.xitu.io/2018/2/10/1617f5c7e2bbd5f7?w=1240&h=531&f=png&s=478335)

总结：Java8其实是一个单体模式，一个简单的HelloWorld，都需要100多M的JRE环境，Java9引入模块后，模块之间依赖关系更加清晰，只需引用需要的模块。

<!--more-->

# 模块化的安全？
+ public不再意味着Accessible：
`requires`：指明对其它模块的依赖
`exports`：控制着哪些包可以被其它模块访问到。所有不被导出的包默认都被封装在模块里面。
所以说先有模块的可读性，进一步才是模块内的可访问性（public）。

+ 模块的间接（Transitive）引用：
比如A模块requires了java.logging模块，B模块requires了A：
如果没有用transitive关键字，那么B模块还需要引入java.logging模块：

```java
module a_module {
    requires java.logging;
}
module b_module {
    requires java.logging;
    requires a_module;
}
```
如果使用了transitive关键字，那么B模块就不需要引入java.logging模块：

```java
module a_module {
    requires java.logging;
}
module b_module {
    requires a_module;
}
```

# module与maven/gradle是什么关系？
模块化的依赖关系，很容易让人联想到mven和gradle，这个在上篇中也提及，后来有读者也提出module和maven是什么关系？解答如下：

Maven 有两个主要的特征：依赖管理和构建管理。
依赖管理即可以决定库中的版本并从仓库中下载下来。
构建管理即管理开发过程中的任务，包括初始化、编译、测试、打包等。

Module是系统内置用于表述组件之间的关系，对于版本的管理还是处于最原始的状体，管理一种强制的依赖关系。

总结一下：Maven还是主要负责构建管理，Modules 对于像Maven这样的构建工具（build tools）来说扮演的是辅助补充的角色。因为这些构建工具在构建时建立的依赖关系图谱和他们版本可以根据Module来创建，Module强制确定了module和artifacts之间的依赖关系，而Maven对于依赖并非是强制的。
具体可参考StackOverflow上的一篇问答：[Project Jigsaw vs Maven on StackOverflow](https://stackoverflow.com/questions/39844602/project-jigsaw-vs-maven)

# 模块化之后如何打包？
+ Jlink：
JLink是将Module打包的工具，打包后的文件非常精简。
使用如下：

```shell
./jlink --module-path ../jmods/ --add-modules java.sql --output /Users/shipeipei/Desktop/xxx
```
![](https://user-gold-cdn.xitu.io/2018/2/10/1617f5c7e463adb4?w=1240&h=28&f=png&s=31588)
执行脚本后，在桌面生成了xxx文件夹，整个文件夹其实就是一个裁剪后的jdk，大小为49.2M：
![](https://user-gold-cdn.xitu.io/2018/2/10/1617f5c8477213e8?w=1240&h=663&f=png&s=251851)

# 模块化的原理？

1、将系统内部类进行模块化

2、将ClassLoader分级：
将ClassLoader分为三个级别：
+ Bootstrap Loader具有最高优先级和权限，主要是核心的系统类；
+ Platform Loader用于扩展的一些系统类，例如SQL,XML等；
+ Application Loader主要用于应用程序的Loader。

在这三个级别的Loader下面有一个统一Module管理，用于控制和管理模块间的依赖关系，可读性，可访问性等。