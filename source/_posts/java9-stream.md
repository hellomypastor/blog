title: Java9新特性系列（Java9新特性系列（Stream改进））
date: 2018-02-27 22:22:32
categories: Java
tags: [Java,Java9新特性]
---
# Java8的Stream

在Java8中，一个比较大的变化就是流（Stream），具体可以看之前的一篇文章：[Java8新特性系列（Stream）](http://hellomypastor.net/2017/12/28/Java8%E6%96%B0%E7%89%B9%E6%80%A7%EF%BC%88Stream%EF%BC%89/)

<!--more-->

# Java9的Stream

Java9中Stream增加了4个方法，分别是：

+ `takeWhile`：在有序的Stream中，takeWhile返回从开头开始的尽量多的元素

```java
default Stream<T> takeWhile(Predicate<? super T> predicate) {
    Objects.requireNonNull(predicate);
    // Reuses the unordered spliterator, which, when encounter is present,
    // is safe to use as long as it configured not to split
    return StreamSupport.stream(
        new WhileOps.UnorderedWhileSpliterator.OfRef.Taking<>(spliterator(), true, predicate),
            isParallel()).onClose(this::close);
}
```

+ `dropWhile`：与`takeWhile`相反

```java
default Stream<T> dropWhile(Predicate<? super T> predicate) {
    Objects.requireNonNull(predicate);
    // Reuses the unordered spliterator, which, when encounter is present,
    // is safe to use as long as it configured not to split
    return StreamSupport.stream(
        new WhileOps.UnorderedWhileSpliterator.OfRef.Dropping<>(spliterator(), true, predicate),
            isParallel()).onClose(this::close);
}
```

+ ofNullable：可以创建一个单元素Stream，可以为null

```java
public static<T> Stream<T> ofNullable(T t) {
    return t == null ? Stream.empty()
         : StreamSupport.stream(new Streams.StreamBuilderImpl<>(t), false);
}
```

+ iterate

```java
public static<T> Stream<T> iterate(final T seed, final UnaryOperator<T> f) {
    Objects.requireNonNull(f);
    Spliterator<T> spliterator = new Spliterators.AbstractSpliterator<>(Long.MAX_VALUE,
    Spliterator.ORDERED | Spliterator.IMMUTABLE) {
        T prev;
        boolean started;

        @Override
        public boolean tryAdvance(Consumer<? super T> action) {
            Objects.requireNonNull(action);
            T t;
            if (started)
                t = f.apply(prev);
            else {
                t = seed;
                started = true;
            }
            action.accept(prev = t);
            return true;
        }
    };
    return StreamSupport.stream(spliterator, false);
}
```

除了对 Stream 本身的扩展，Optional和Stream之间的结合也得到了改进，可以将optional对象转化为stream对象：

```java
List<String> list = new ArrayList<>() {{
    add("a");add("b");add("c");
}};
Optional<List<String>> optional = Optional.ofNullable(list);
Stream<List<String>> stream = optional.stream();
stream.flatMap(x -> x.stream()).forEach(System.out::println);
```

# 使用举例

+ takeWhile

```java
List<Integer> list = Arrays.asList(1, 4, 5, 2, 3, 6, 7, 8, 9, 10);
list.stream().takeWhile(x -> x < 5).forEach(System.out::println);
```
>输出：<br>1<br>4

+ dropWhile

```java
List<Integer> list = Arrays.asList(1, 4, 5, 2, 3, 6, 7, 8, 9, 10);
list.stream().dropWhile(x -> x < 5).forEach(System.out::println);
```
>输出：<br>5<br>2<br>3<br>6<br>7<br>8<br>9<br>10

+ ofNullable

```java
Stream<String> stream = Stream.of("", null);
System.out.println(stream.count());

Stream<String> stream = Stream.of(null);
System.out.println(stream.count());//会抛空指针异常

Stream<String> stream = Stream.ofNullable(null);
System.out.println(stream.count());
```
>输出：<br>2<br>0

+ iterate

```java
//java8
Stream.iterate(1, i -> i + 1).limit(5).forEach(System.out::println);

//java9
Stream.iterate(1, i -> i < 6, i -> i + 1).forEach(System.out::println);
```
>输出：<br>1<br>2<br>3<br>4<br>5