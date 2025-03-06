title: Java9新特性系列（<>钻石操作符改进）
date: 2018-02-23 22:16:31
categories: Java
tags: [Java,Java9新特性]
---
# Java7前时代

在Java7之前每次声明泛型变量的时必须左右两边都同时声明泛型：

```java
List<String> list = new ArrayList<String>();
Set<String> set = new HashSet<String>();
Map<String, List<String>> map = new HashMap<String, List<String>>();
```
>这样看来右边的泛型声明就变得是多余的了？

# Java7

在Java7中，对这一点进行了改进，就不必两边都要声明泛型，这种只适用<>标记的操作，称之为`钻石操作符Diamond Operator`：

```java
List<String> list = new ArrayList<>();
Set<String> set = new HashSet<>();
Map<String, List<String>> map = new HashMap<>();
```
>对比之前的用法是不是很清晰很方便呢？

但是Java7中钻石操作符不允许在匿名类上使用：

```java
List<String> list = new ArrayList<>();
List<String> list = new ArrayList<>(){};//报错
Set<String> set = new HashSet<>();
Set<String> set = new HashSet<>(){};//报错
Map<String, List<String>> map = new HashMap<>();
Map<String, List<String>> map = new HashMap<>(){};//报错
```
>如果与匿名类共同使用，会报错：`'<>' cannot be used with anonymous classes`

<!--more-->

# Java9

在Java9中，钻石操作符能与匿名实现类共同使用，[官方Feature](http://openjdk.java.net/jeps/213)给出了如下说明：
>**Allow diamond with anonymous classes if the argument type of the inferred type is denotable.** Because the inferred type using diamond with an anonymous class constructor could be outside of the set of types supported by the signature attribute, using diamond with anonymous classes was disallowed in Java SE 7. As noted in the JSR 334 proposed final draft, it would be possible to ease this restriction if the inferred type was denotable.

```java
List<String> list = new ArrayList<>() {
    @Override
    public int size() {
        return super.size();
    }

    @Override
    public String toString() {
        return super.toString();
    }
};
        
Set<String> set = new HashSet<>() {
    @Override
    public int size() {
        return super.size();
    }
    
    @Override
    public String toString() {
        return super.toString();
    }
};

Map<String, List<String>> map = new HashMap<>() {
    @Override
    public int size() {
        return super.size();
    }

    @Override
    public String toString() {
        return super.toString();
    }
};
```