title: Java9新特性系列（module&spi）
date: 2018-02-13 22:32:30
categories: Java
tags: [Java,Java9新特性]
---
上两篇已经深入分析了[Java9新特性系列（深入理解模块化）](http://hellomypastor.net/2018/02/10/Java9%E6%96%B0%E7%89%B9%E6%80%A7%E7%B3%BB%E5%88%97%EF%BC%88%E6%B7%B1%E5%85%A5%E7%90%86%E8%A7%A3%E6%A8%A1%E5%9D%97%E5%8C%96%EF%BC%89/)，以及[Java9新特性系列（module&maven&starter）](http://hellomypastor.net/2018/02/11/Java9%E6%96%B0%E7%89%B9%E6%80%A7%E7%B3%BB%E5%88%97%EF%BC%88module-maven-starter%EF%BC%89/)，有读者又提到了与模块化相关的`spi`，本篇将进行分析。
# SPI是什么？
提到SPI呢，就不得不提一下API：

API：Application Programming Interface，即应用程序编程接口，在程序外部进行调用

SPI：Service Provider Interface，服务提供商接口

# SPI核心思想是什么？
我们知道，一个系统中会有很多模块，比如数据库模块、日志模块、调度模块、各种业务模块等等，每一类的模块都有很多种实现，
数据库可以用mysql、oracle等，日志可以用log4j、logback等，那么对于不同的场景有不同的选型，如何能做到可插拔呢，那就是SPI了。
>可插拔原则可以理解为系统与插件的关系，系统提供了一些接口，第三方插件进行实现。

面向接口编程，不绑定实现，为了在模块装配的时候不在代码中动态去指定具体的实现，就需要去发现具体的实现，即服务发现，其实就类似于回调，只不过回调的时候需要找到具体的实现，spi就帮我们做了去寻找实现的工作。
>这一思想在模块化设计中尤为重要。

# SPI实现方式？
SPI具体的实现方式分两种：
+ 应用系统自身提供默认实现
+ 第三方提供实现

<!--more-->

# SPI有什么例子呢？
>下面我们就以jdk中的jdbc模块儿进行分析：

```java
package java.sql;

import java.util.logging.Logger;

public interface Driver {
    Connection connect(String url, java.util.Properties info)
        throws SQLException;
    
    boolean acceptsURL(String url) throws SQLException;
    
    DriverPropertyInfo[] getPropertyInfo(String url, java.util.Properties info)
                         throws SQLException;
    
    int getMajorVersion();
    
    int getMinorVersion();
    
    boolean jdbcCompliant();
    
    public Logger getParentLogger() throws SQLFeatureNotSupportedException;
}
```
>我们可以看到，java.sql.Driver接口定义了每个实现必须实现的一些方法，接下来我们就看具体的实现：

+ mysql

META-INF/services/java.sql.Driver

```
com.mysql.jdbc.Driver
com.mysql.fabric.jdbc.FabricMySQLDriver
```
在这个文件中指定了具体的实现

+ oracle

META-INF/services/java.sql.Driver

```
oracle.jdbc.OracleDriver
```
>在JDBC4以前，我们还需要使用比如Class.forName("com.mysql.jdbc.Driver")的方式来装载驱动。<br><br>如上图所示：<br>JDBC也基于spi的机制来发现驱动提供商了，可以通过`META-INF/services/java.sql.Driver`文件里指定实现类的方式来暴露驱动提供者。<br><br>其中，`META-INF/services/`是固定的，`java.sql.Driver`为接口对应的`package`，文件中为具体的实现类。

# SPI如何被框架发现呢？
>框架可以使用java提供的java.util.ServiceLoader类得到SPI的实现。我们来看下java.sql.DriverManager中是如何去发现的：

```java
private static void loadInitialDrivers() {
    ...
    AccessController.doPrivileged(new PrivilegedAction<Void>() {
        public Void run() {
            // 获取具体的实现类
            ServiceLoader<Driver> loadedDrivers = ServiceLoader.load(Driver.class);
            Iterator<Driver> driversIterator = loadedDrivers.iterator();
            try{
                while(driversIterator.hasNext()) {
                    driversIterator.next();
                }
            } catch(Throwable t) {
                // Do nothing
            }
            return null;
        }
    });
    
    ...
    
    for (String aDriver : driversList) {
        try {
            println("DriverManager.Initialize: loading " + aDriver);
            Class.forName(aDriver, true,
                    ClassLoader.getSystemClassLoader());
        } catch (Exception ex) {
            println("DriverManager.Initialize: load failed: " + ex);
        }
    }
}
```
>使用`ServiceLoader<XXX> loadedDrivers = ServiceLoader.load(XXX.class);`，实际上就是到具体的路径下读取文件内容。

# SPI应用
用过阿里`Dubbo`的开发者都知道，Dubbo对JDK中SPI进行了扩展和改进，这个在以后dubbo相关的文章中再进行介绍~


>好了，本期就讲到这里，下期我们讲讲Java9中到另一个新工具`JShell`，敬请期待～