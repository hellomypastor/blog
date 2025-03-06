title: Java9新特性系列（便利的集合工厂方法）
date: 2018-02-25 22:19:23
categories: Java
tags: [Java,Java9新特性]
---
# Java8前时代

>在Java8版本以前，创建一个**只读不可变**的集合，先要初始化，然后塞数据，然后置为只读：

```java
Set<String> set = new HashSet<>();
set.add("a");
set.add("b");
set.add("c");
set = Collections.unmodifiableSet(set);
```

上面的方式占用太多行，能不能用单行表达式呢？用如下方式：

```java
Set<String> set = Collections.unmodifiableSet(new HashSet<>(Arrays.asList("a", "b", "c")));
Set<String> set = Collections.unmodifiableSet(new HashSet<String>() {{
    add("a"); add("b"); add("c");
}});
```

# Java8

在Java8中可以用流的方法创建，具体可以看之前的一篇文章[Java8新特性系列（Stream）](http://hellomypastor.net/2017/12/28/Java8%E6%96%B0%E7%89%B9%E6%80%A7%EF%BC%88Stream%EF%BC%89/)，实现方法如下：

```java
Set<String> set = Collections.unmodifiableSet(Stream.of("a", "b", "c").collect(toSet()));
```
<!--more-->

# Java9

Java9中引入了很多方便的API，Convenience Factory Methods for Collections，即集合工厂方法，[官方Feature](http://openjdk.java.net/jeps/269)，上述的实现中Java9中有如下实现方式：

```java
Set<String> set = Collections.unmodifiableSet(new HashSet<String>() {{
    add("a"); add("b"); add("c");
}});
```

也可以用如下方式：

```java
Set<String> set = Set.of("a", "b", "c");
```

Java9中`List`提供了一系列类似的方法：

```java
/**
 * Returns an immutable list containing zero elements.
 * @since 9
 */
static <E> List<E> of() {
    return ImmutableCollections.List0.instance();
}

/**
 * Returns an immutable list containing one element.
 * @since 9
 */
static <E> List<E> of(E e1) {
    return new ImmutableCollections.List1<>(e1);
}

/**
 * Returns an immutable list containing one element.
 * @since 9
 */
static <E> List<E> of(E e1) {
    return new ImmutableCollections.List1<>(e1);
}

/**
 * Returns an immutable list containing three elements.
 * @since 9
 */
static <E> List<E> of(E e1, E e2, E e3) {
    return new ImmutableCollections.ListN<>(e1, e2, e3);
}

...

/**
 * Returns an immutable list containing ten elements.
 * @since 9
 */
static <E> List<E> of(E e1, E e2, E e3, E e4, E e5, E e6, E e7, E e8, E e9, E e10) {
    return new ImmutableCollections.ListN<>(e1, e2, e3, e4, e5, e6, e7, e8, e9, e10);
}

@SafeVarargs
@SuppressWarnings("varargs")
static <E> List<E> of(E... elements) {
    switch (elements.length) { // implicit null check of elements
        case 0:
            return ImmutableCollections.List0.instance();
        case 1:
            return new ImmutableCollections.List1<>(elements[0]);
        case 2:
            return new ImmutableCollections.List2<>(elements[0], elements[1]);
        default:
            return new ImmutableCollections.ListN<>(elements);
    }
}
```

Java9中Set、Map都有类似的方法，创建**只读不可变**的集合：

```java
Set.of()
...
Map.of()
Map.of(k1, v1)
Map.of(k1, v1, k2, v2)
Map.of(k1, v1, k2, v2, k3, v3)
...
Map.ofEntries(Map.Entry<K,V>...)
Map.Entry<K,V> entry(K k, V v)
Map.ofEntries(
    entry(k1, v1),
    entry(k2, v2),
    entry(k3, v3),
    // ...
    entry(kn, vn)
);
```