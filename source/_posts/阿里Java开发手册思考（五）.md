title: 阿里Java开发手册思考（五）
date: 2017-12-17 21:21:34
categories: Java
tags: [Java,阿里Java开发手册]
---
>上期我们分享了Java中日志的处理（下）：Java中日志实际使用中的相关注意点
>
>本期我们将分享Java中异常的处理（上）

<!--more-->
##异常定义
在《java编程思想》中这样定义异常：阻止当前方法或作用域继续执行的问题。
##异常分类
首先我们看下Java中异常的继承关系：

可以看出，`Throwable`有两个子类：`Error`和`Exception`

+ `Error`
    + VirtualMachineError，典型的有`StackOverFlow`和`OutOfMemory`
    + AWTError
+ `Exception`
    + IOException
    + ...
    + RuntimeException

Exception分为CheckedException和UncheckedException，那么CheckedException和UncheckedException区别是什么呢？

+ `UncheckedException`：派生于Error或者RuntimeException的异常
+ `CheckedException`：所有其他的异常

##异常处理机制
异常处理机制分为：抛出异常和捕捉异常

抛出异常：方法上使用`throws`，方法内使用`throw`
捕捉异常：使用`try-catch`或者`try-catch-finally`

原则，正如手册上所说：

+ 不要直接忽略异常
+ 不要用try-catch包住太多语句
+ 不要用异常处理来处理程序的正常控制流
+ 不要随便将异常迎函数栈向上传递，能处理尽量处理

何时向上传播？

+ 当你认为本异常应该由上层处理时，才向上传播

##注意点
+ `finally`语句块一定会执行吗？

>**不一定会**，以下两种情况finally语句块不会执行
>1. 未执行到try语句块
>2. try语句块中有System.exit(0);  

+ `finally`语句块的执行顺序

首先看没有控制语句的情况：

```java
public static void main(String[] args) {
	try {
		System.out.println("try block");
	} finally {
		System.out.println("finally block");
	}
}
```
输出没有疑问：
try block
finally block

>1、如果`try`中有控制语句（`return`、`break`、`continue`），那`finally`语句块是在控制转义语句之前执行还是之后执行？

```java
private static String test1() {
	System.out.println("test1()");
	return "return";
}

private static String test() {
	try {
		System.out.println("try block");
		return test1();
	} finally {
		System.out.println("finally block");
	}
}

public static void main(String[] args) {
	System.out.println(test());
}
```
输出：
try block
test1()
finally block
return

所以说，如果`try`中有控制语句（`return`、`break`、`continue`），那`finally`语句块是在控制转义语句**之前**执行

>2、如果`catch`语句中有控制语句（`return`、`break`、`continue`），那`finally`语句块是在控制转义语句之前执行还是之后执行？

```java
private static String test1() {
	System.out.println("test1()");
	return "return";
}

private static String test() {
	try {
		System.out.println("try block");
		System.out.println(1 / 0);
		return test1();
	} catch (Exception e) {
		System.out.println("catch block");
		return test1();
	} finally {
		System.out.println("finally block");
	}
}

public static void main(String[] args) {
	System.out.println(test());
}
```

输出：
try block
catch block
test1()
finally block
return

所以说，如果`catch`语句中有控制语句（`return`、`break`、`continue`），那`finally`语句块是在控制转义语句**之前**执行

+ `finally`里的变量

```java
public static int test() {
	int i = 0;
	try {
		return i;
	} finally {
		i++;
	}
}

public static void main(String[] args) {
	System.out.println(test());
}
```

输出：
0

咦？很奇怪，为什么是0，而不是1呢？

通过反编译生成的class，我们就能知道原因了

```java
int i = 0;
try {
	return i;
} finally {
	int iTemp = i++;
}
```

原来，i++后只是赋值给了一个新的`局部变量`，i本身并没有变，这一点和函数的形参一样，如果传的是`引用类型`的，那么值会变，如果传的不是引用类型，那么值是不会改变的，改变的也只是局部变量。

##常见面试题
+ 1、error和exception有什么区别？

>error 表示系统级的错误，是java运行环境内部错误或者硬件问题，不能指望程序来处理这样的问题，除了退出运行外别无选择，它是Java虚拟机抛出的。
exception 表示程序需要捕捉、需要处理的异常，是由与程序设计的不完善而出现的问题，程序必须处理的问题

+ 2、运行时异常和一般异常有何不同

>Java 提供了两类主要的异常：runtimeException 和 checkedException
一般异常（checkedException）主要是指 IO 异常、SQL 异常等。对于这种异常，JVM 要求我们必须对其进行 cathc 处理，所以，面对这种异常，不管我们是否愿意，都是要 写一大堆的 catch 块去处理可能出现的异常。
运行时异常（runtimeException）我们一般不处理，当出现这类异常的时候程序会由虚拟机接管。比如，我们从来没有去处理过 NullPointerException，而且这个异常还是最 常见的异常之一。
出现运行时异常的时候，程序会将异常一直向上抛，一直抛到遇到处理代码，如果没有 catch 块进行处理，到了最上层，如果是多线程就有 Thread.run()抛出，如果不是多线程 那么就由 main.run() 抛出。抛出之后，如果是线程，那么该线程也就终止了，如果是主程序，那么该程序也就终止了。
其实运行时异常的也是继承自 Exception，也可以用 catch 块对其处理，只是我们一般不处理罢了，也就是说，如果不对运行时异常进行 catch 处理，那么结果不是线程退出就是 主程序终止。
如果不想终止，那么我们就必须捕获所有可能出现的运行时异常。如果程序中出现了异常数据，但是它不影响下面的程序执行，那么我们就该在catch块里面将异常数据舍弃， 然后记录日志。如果，它影响到了下面的程序运行，那么还是程序退出比较好些。

+ 3、Java 中异常处理机制的原理

>Jav a通过面向对象的方式对异常进行处理，Java 把异常按照不同的类型进行分类，并提供了良好的接口。在 Java 中，每个异常都是一个对象，它都是 Throwable 或其子类的实例。当一个方法出现异常后就会抛出一个异常对象，该对象中包含有异常信息，调用这个对象的方法可以捕获到这个异常并对异常进行处理。Java 的异常处理是通过5个 关键词来实现的：try、catch、throw、throws、finally。
try：用来指定一块预防所有异常的程序
catch：紧跟在try后面，用来捕获异常
throw：用来明确的抛出一个异常
throws：用来标明一个成员函数可能抛出的各种异常
finally：确保一段代码无论发生什么异常都会被执行的一段代码。

+ 4、你平时在项目中是怎样对异常进行处理的。
>1）尽量避免出现 runtimeException。例如对于可能出现空指针的代码，带使用对象之前一定要判断一下该对象是否为空，必要的时候对 runtimeException也进行 try catch 处理。
2）进行 try catch 处理的时候要在 catch 代码块中对异常信息进行记录，通过调用异常类的相关方法获取到异常的相关信息。

+ 5、final、finally、finalize的区别
>（1）、final 用于声明变量、方法和类的，分别表示变量值不可变，方法不可覆盖，类不可以继承
（2）、finally 是异常处理中的一个关键字，表示 finally{} 里面的代码一定要执行
（3）、finalize 是 Object 类的一个方法，在垃圾回收的时候会调用被回收对象的此方法。