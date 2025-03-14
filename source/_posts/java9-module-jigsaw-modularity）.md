title: 'Java9新特性系列（模块化系统: Jigsaw->Modularity）'
date: 2018-01-31 23:08:33
categories: Java
tags: [Java,Java9新特性]
---
# 模块化的前时代
`Ant时代`
相信大家对Ant都不陌生，Ant是任务型的，定义了一系列的任务dir／compile／jar等等，缺点是操作文件
`Maven时代`
Maven／Gradle相对于Ant，增加了groupId／artifactId，版本管理，依赖管理
`OSGI时代`
OSGI用的比较少，估计大家比较陌生，其实我们用的Eclipse及插件都是OSGI编译的

>那么问题来了：当代码库越来越大，怎么管理类库交叉依赖，以及类重复的问题呢？

<!--more-->
# 产生背景及意义
+ 谈到Java9大家往往第一个想到的就是Jigsaw项目。众所周知，Java已经发展超过20年(95年最初发布)，Java和相关生态在不断丰富的同时也越来越暴露出一些问题:
    + `Java运行环境的膨胀和臃肿`。每次JVM启动的时候，至少会有30~60MB的内存加载，主要原因是JVM需要加载rt.jar，不管其中的类是否被classloader加载，第一步整个jar都会被JVM加载到内存当中去(而模块化可以根据模块的需要加载程序运行需要的class)
    + `当代码库越来越大，创建复杂，盘根错节的“意大利面条式代码”的几率呈指数级的增长`。不同版本的类库交叉依赖导致让人头疼的问题，这些都阻碍了Java开发和运行效率的提升
    + `很难真正地对代码进行封装, 而系统并没有对不同部分(也就是JAR文件)之间的依赖关系有个明确的概念`。每一个公共类都可以被类路径之下任何其它的公共类所访问到，这样就会导致无意中使用了并不想被公开访问的API。
    + `类路径本身也存在问题`: 你怎么知晓所有需要的JAR都已经有了, 或者是不是会有重复的项呢?

+ 同时，由于兼容性等各方面的掣肘，对Java进行大刀阔斧的革新越来越困难，Jigsaw从Java7阶段就开始筹备，Java8阶段进行了大量工作，`终于在Java9里落地，一种千呼万唤始出来的意味`。

+ Jigsaw项目(后期更名为Modularity)的工作量和难度大大超出了初始规划。JSR376 Java平台模块化系统(JPMS,Java Platform Module System)作为Jigsaw项目的核心, 其主体部分被分解成6个JEP(JDK Enhancement Proposals)。

+ 作为Java9平台最大的一个特性，随着Java平台模块化系统的落地，开发人员无需再为不断膨胀的 Java平台苦恼，例如，您可以使用**`jlink`工具，根据需要定制运行时环境**。这对于拥有大量镜像的容器应用场景或复杂依赖关系的大型应用等，都具有非常重要的意义。

+ 本质上讲，模块(module)的概念，其实就是package外再裹一层，也就是说，用模块来管理各个package，通过声明某个package暴露，不声明默认就是隐藏。因此，**`模块化使得代码组织上更安全，因为它可以指定哪些部分可以暴露，哪些部分隐藏`**。

# 设计理念
`模块独立，化繁为简`
模块化(以 Java 平台模块系统的形式)将JDK分成一组模块，可以在编译时，运行时或者构建时进行组合。

# 实现目标
+ 主要目的在于减少内存的开销
+ 只须必要模块，而非全部jdk模块，可简化各种类库和大型应用的
开发和维护
+ 改进Java SE平台，使其可以适应不同大小的计算设备
+ 改进其安全性，可维护性，提高性能