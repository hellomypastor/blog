title: Java9新特性系列（HTTP 2 Client）
date: 2018-02-28 22:25:16
categories: Java
tags: [Java,Java9新特性]
---
# HTTP

HTTP，HyperText Transfer Protocol，即超文本传输协议。1999年6月定义了HTTP协议中现今广泛使用的一个版本——HTTP 1.1，HTTP/2标准于2015年5月正是发表。

# HTTP/2与HTTP1.1的区别

+ HTTP/2采用二进制格式而非文本格式
+ 完全的多路复用
+ 使用头压缩，降低了开销
+ 让服务器可以主动推送（push）数据到客户端

<!--more-->

# Java9中新的Http Client

[官方Feature](http://openjdk.java.net/jeps/110)：

### 动机
>The existing HttpURLConnection API and its implementation have numerous problems:
* The base URLConnection API was designed with multiple protocols in mind, nearly all of which are now defunct (ftp, gopher, etc.).
* The API predates HTTP/1.1 and is too abstract.
* It is hard to use, with many undocumented behaviors.
* It works in blocking mode only (i.e., one thread per request/response).

+ It is very hard to maintain.

### 摘要

>Define a new HTTP client API that implements HTTP/2 and WebSocket, and can replace the legacy HttpURLConnection API. The API will be delivered as an incubator module, as defined in JEP 11, with JDK 9. This implies:
* The API and implementation will not be part of Java SE.
* The API will live under the jdk.incubtor namespace.
* The module will not resolve by default at compile or run time.

### 总结

在Java9之前，都是使用`HttpURLConnection`，本身存在很多问题，比如只支持HTTP/1.1、很难用、只适用于blocking模式、很难维护等等。
在Java9中，提供了新的方式来处理HTTP调用，提供了新的HTTP Client，将替代HttpURLConnection，并提供对`WebSocket`和`HTTP/2`的支持，且提供流和服务器推送等功能的API。

新的HTTP Client在jdk.incubtor.httpclient模块中，如果要使用，需要在模块声明文件(module-info.java)中引入：

```java
module main_module {
    ...
    requires jdk.incubtor.httpclient;
    ...
}
```

### 使用举例

```java
HttpClient client = HttpClient.newHttpClient(); 
HttpRequest req = HttpRequest.newBuilder(URI.create("http://hellomypastor.net" )).GET().build();
HttpResponse<String> response = client.send(req, HttpResponse.BodyHandler.asString());
System.out.println(response.statusCode());
System.out.println(response.version().name());
System.out.println(response.body());
}
```