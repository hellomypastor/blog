title: Stream.peek() method in Java 8 vs Java 9
date: 2018-10-17 20:42:16
categories: Java
tags: [Java,Java9新特性]
---
最近从 `Oracle Jdk8` 切换到 `OpenJdk 10` 后，遇到了一个诡异的问题，通过 Debug 排查后发现原来是 `Stream.peek()` 方法在搞鬼，特此记录下，以防后续使用 `Java 9及以上` 版本时再次犯同样的错误。

示例代码如下：

```java
public static void main(String[] args) {
    List<String> list = new ArrayList<>() {{
        add("a");
        add("b");
        add("c");
    }};

    list.stream().peek(System.out::println).count();
}
```

在 `Java 8` 中，输出应如下：

```java
a
b
c
```

切换到 `Java 9` 后，没有任何输出!!!

突然有点怀疑人生的感觉-_-#

<!--more-->

于是就去 `StackOverFlow` （如果你不知道的话，赶紧去访问看看～）问答网站上搜索，发现也有人出现一样的问题：[Stream.peek() method in Java 8 vs Java 9](https://stackoverflow.com/questions/48221783/stream-peek-method-in-java-8-vs-java-9)

针对以上问题，分析如下：

#### Stream.peek() 方法介绍

生成一个包含原 `Stream` 的所有元素的新 Stream，同时会提供一个消费函数（ `Consumer` 实例），新 Stream 每个元素被消费的时候都会执行给定的消费函数；

代码定义如下：

```java
Stream<T> peek(Consumer<? super T> action);
```

示意图如下：

![](https://user-gold-cdn.xitu.io/2018/10/17/166821db0803fcbd?w=403&h=212&f=jpeg&s=33878)

#### Java 9 中 Stream.peek() 不执行的原因

`Javadoc` 关于 `Stream` 说明如下：

>A stream implementation is permitted significant latitude in optimizing the computation of the result. For example, `a stream implementation is free to elide operations (or entire stages) from a stream pipeline -- and therefore elide invocation of behavioral parameters -- if it can prove that it would not affect the result of the computation`. This means that `side-effects of behavioral parameters may not always be executed` and should not be relied upon, unless otherwise specified (such as by the terminal operations forEach and forEachOrdered). (For a specific example of such an optimization, see the API note documented on the count() operation. For more detail, see the side-effects section of the stream package documentation.)

`Javadoc` 关于 `count()` 说明如下：
>API Note:<br>
`An implementation may choose to not execute the stream pipeline (either sequentially or in parallel) if it is capable of computing the count directly from the stream source`. In such cases no source elements will be traversed and `no intermediate operations will be evaluated`. Behavioral parameters with side-effects, which are strongly discouraged except for harmless cases such as debugging, may be affected. 

从 `Javadoc` 中可以知道，当 `List` 的元素个数没有变化时，没必要执行 `map` 和 `peek` 方法，而且`终止操作`为 `count()`，元素个数没有变化时，也没必要执行 `map` 和 `peek` 方法，除非使用能影响元素个数的一些方法，如 `filter` 和 `distinct`，所以在 `Java 9` 中使用 `Stream.peek()` 与 `Stream.map()` 方法要注意使用场景，否则方法是不会执行的。

`Javadoc` 中还有一段说明，`peek()` 方法主要用于 `debug`：

>API Note <br>This method exists mainly to `support debugging`, where you want to see the elements as they flow past a certain point in a pipeline.