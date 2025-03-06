title: Spring Test & Junit单元测试
date: 2017-08-14 23:12:36
categories: Java
tags: [Java,单元测试]
---
只要做过java web的人一定都接触过单元测试，单元测试是代码写完之后的第一道工序，是提高代码质量必不可少的途径～
大多数java web项目都是基于Spring框架来做的，必然涉及到持久化，如数据库操作，那么对于有数据库操作的单元测试应该如何写呢？
<!--more-->
第一个我们想到的就是`Mockito`，就是说模拟数据库操作，模拟输入模拟输出，这样能将精力放在业务逻辑（service）层的测试上，与之相关的还有`PowerMockito`以及`MockMVC`，那么如果想对数据库操作进行单元测试呢？
首先，我们要初始化`ApplicationContext`，在before方法中去初始化ApplicationContext，可能会导致初始化多次，这显然不妥～
初始化之后，dao层对数据库进行操作时，会对数据库进行操作，可能会影响数据库的数据，对数据造成破坏，违背了对数据库现场不破坏的原则，那么针对这两个问题，我们应该如何解决呢？请看如下代码：
首先定义一个单元测试基类，所有的单元测试类需继承此类
```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration({ "classpath*:app*.xml" })
public class BaseJunit4Test {
  //
}
```
`@RunWith(SpringJUnit4ClassRunner.class)`表示使用junit4进行测试
`@ContextConfiguration({ "classpath*:app*.xml" })`表示加载配置文件

单元测试类如下：
```java
public class ServiceTest extends BaseJunit4Test {
  @Resource
  private CityService cityService;
  @Test
  @Transactional
  @Rollback(true)
  public void doTest() {
    cityService.delete(1);
  }
}
```
`@Transactional`表示使用事务
`@Rollback`表示是否回滚

运行后，成功，但是delete方法执行后并没有删除数据库中的数据，符合我们的预期

通过这样的方法，Sping Test+Junit，我们就能方便的对dao层进行单元测试，而不影响数据库现场