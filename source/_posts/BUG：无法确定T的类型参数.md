title: BUG：无法确定T的类型参数
date: 2017-07-31 22:30:46
type: "tags"
tags:
- Java
categories: 疑难杂症
---
最近遇到了与jdk版本有关的问题，同样的代码在本地跑起来没有问题，一到服务器上就出问题了，问题如下：
无法确定T的类型参数；对于上下限为T，java.lang.Object的类型变量下，不存在唯一最大实例，乍一看，类型转换出错了？
<!--more-->
## 废话不多说，先上代码：
```java
private MyBean bean;
public void test()
{
  WhiteBox.setIntervalState(bean, 'id', 1);
  TestCase.assertTrue(bean.getId() == 1);
}
```
## 运行时，出现如下错误：
```js
无法确定T的类型参数；对于上下限为T，java.lang.Object的类型变量下，不存在唯一最大实例
```
## 解决方法
那么如何解决呢？不存在唯一最大实例，那么就强制类型转换好了，修改如下：
```java
Integer id = Integer.valueOf(bean.getId());
```
## 原因分析
通过对比发现，服务器的jdk版本为`jdk1.6.0_03`，而本地的jdk版本为`jdk1.6.0_45`，通过上网搜索发现，这是jdk的一个bug，具体链接为[jdk bug 6468354](https://blogs.openjdk.java.net/browse/JDK-6468354)，这个bug已在jdk1.6.0_25修复，所以本地的运行没有问题，服务器上的运行有问题。

小伙伴们，下次如果遇到同样的问题，记得看下jdk的版本号哦～