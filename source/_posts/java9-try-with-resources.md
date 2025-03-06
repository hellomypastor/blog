title: Java9新特性系列（try-with-resources改进）
date: 2018-02-22 22:13:56
categories: Java
tags: [Java,Java9新特性]
---
# Java7前时代的try
>在Java7版本以前，try的使用方法如下（流等资源的关闭在finally中进行，否则会导致资源泄漏）：

```java
public static void main(String[] args) {
    InputStreamReader reader = null;
    try {
        reader = new InputStreamReader(System.in);
        ...
        reader.read();
        ...
    } catch (IOException e) {
        e.printStackTrace();
    } finally {
        if (reader != null) {
            try {
                reader.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```

# Java7
Java7中，可以实现资源的自动关闭，但前提是资源必须要在try的子语句中进行初始化，否则编译会报错：

```java
public static void main(String[] args) {
    try (InputStreamReader reader = new InputStreamReader(System.in)) {
        ...
        reader.read();
        ...
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```
Java7中出了try作了改进，catch也同样作了改进：
+ Java7之前

```java
try {
    //逻辑代码
    ...
} catch (IOException ex) {
    logger.log(ex);
} catch (SQLException ex) {
    logger.log(ex);
}
...
```

+ Java7及Java7以上

```java
try {
    //逻辑代码
    ...
} catch (IOException | SQLException ex) {
    logger.log(ex);
}
...
```
>catch子语句中ex默认是final的，在catch语句块中不能改变ex，否则会编译报错。

<!--more-->

# Java9
Java9中try更加灵活强大，支持在try子语句外部定义resource，[官方Feature](http://openjdk.java.net/jeps/213)给出了如下说明：
>Allow effectively-final variables to be used as resources in the try-with-resources statement. The final version of try-with-resources statement in Java SE 7 requires a fresh variable to be declared for each resource being managed by the statement. This was a change from earlier iterations of the feature. The public review draft of JSR 334 discusses the rationale for the change from the early draft review version of try-with-resource which allowed an expression managed by the statement. **The JSR 334 expert group was in favor of an additional refinement of try-with-resources: if the resource is referenced by a final or effectively final variable, a try-with-resources statement can manage the resource without a new variable being declared.** This restricted expression being managed by a try-with-resources statement avoids the semantic issues which motivated removing general expression support. At the time the expert group settled on this refinement, there was insufficient time in the release schedule to accommodate the change.

```java
public static void main(String[] args) {
    InputStreamReader reader = new InputStreamReader(System.in);
    try (reader) {
        ...
        reader.read();
        ...
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```

>如果放在try子语句中，那么reader默认是final的，不可以重新赋值，如果重新赋值，那么编译会报错：`Variable used as a try-with-resource resource should be final or effectively final.`