title: Java9新特性系列（Interface改进）
date: 2018-02-21 22:06:58
categories: Java
tags: [Java,Java9新特性]
---
# Java8前时代的Interface
>在Java8版本以前，Interface接口中所有的方法都是`抽象方法`和`常量`

# Java8
Java8接口相关的可以看之前的一篇文章[Java8新特性系列（Interface）](http://hellomypastor.net/2017/12/21/Java8%E6%96%B0%E7%89%B9%E6%80%A7%EF%BC%88Interface%EF%BC%89/)

## 静态成员
>在Java8中Interface支持`静态成员`，成员默认是`public final static`的，可以在类外直接调用。

## default函数
>在Java8中，Interface中支持函数有实现，只要在函数前加上`default`关键字即可，如下：

## static函数
>在Java8中允许Interface定义`static`方法，这允许API设计者在接口中定义像getInstance一样的静态工具方法，这样就能够使得API简洁而精练。

## @FunctionalInterface注解
+ 什么是`函数式接口`？

>函数式接口其实本质上还是一个接口，但是它是一种特殊的接口：SAM类型的接口（Single Abstract Method）。定义了这种类型的接口，使得以其为参数的方法，可以在调用时，使用一个`Lambda`表达式作为参数。`@FunctionalInterface`注解能帮我们检测Interface是否是函数式接口，但是这个注解**是非必须的，不加也不会报错**。

+ 函数式接口的作用？

>函数式接口，可以在调用时，使用一个lambda表达式作为参数。

Java8中的Interface扩展了Java8之前的接口，更像是一个抽象类。

<!--more-->

# Java9
Java9中Interface更加灵活强大，支持私有方法，[官方Feature](http://openjdk.java.net/jeps/213)给出了如下说明：
>**Support for private methods in interfaces** was briefly in consideration for inclusion in Java SE 8 as part of the effort to add support for Lambda Expressions, but was withdrawn to enable better focus on higher priority tasks for Java SE 8. It is now proposed that support for private interface methods be undertaken thereby enabling non abstract methods of an interface to share code between them.

```java
package net.hellomypastor.java9;

@FunctionalInterface
public interface MyInterface {
    void test();

    default void defaultMethod() {
        privateMethod();
        System.out.println("defaultMethod");
    }

    public static void staticMethod() {
        System.out.println("staticMethod");
    }

    private void privateMethod() {
        System.out.println("privateMethod");
    }
}
```

>接口中的私有方法供内部调用，不会暴露给实现类。