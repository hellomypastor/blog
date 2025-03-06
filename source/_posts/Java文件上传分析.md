title: Java文件上传分析
date: 2017-08-03 23:55:04
type: "tags"
tags:
- Java
---
只要做过java web的人一定都接触过文件上传，但是中间或多或者肯定遇到过问题，那么下面就来分析下文件上传中遇到的各种坑～
<!--more-->
##struts2表单上传文件
struts是通过默认拦截器实现的：
```xml
<interceptor name="fileUpload" class="org.apache.struts2.interceptor.FileUploadInterceptor"/>
```
struts上传文件大小默认最大为2M（default.properties）：
```properties
struts.multipart.maxSize=2097152
```
当然我们也可以自定义设置（struts.properties）：
```properties
struts.multipart.maxSize=209715200
```
也可以在xml进行配置：
```xml
<constant name="struts.multipart.maxSize" value="209715200">
</constant>
```
前端实现：
```html
<form action="xxx" enctype="multipart/form-data" method="post">
...
</form>
```
后台action实现：
```java
private File file;
private String fileFileName;
private String fileContentType
```
后台通过这样的写法，struts框架会自动赋值给这三个属性
##spring  mvc表单上传文件
前端同上
后台实现：
```java
private MultiFile file;
```
上述两种都是通过form表单提交的，那么如何通过ajax直接上传文件呢？
##spring mvc+ajax上传文件
后台实现同上，但是如果不另加配置，则直接报415错误，需要加如下配置：
```xml
<bean class="org.springframework.web.servlet.mvc.annotation.AnnotationMethodHandlerAdapter">
    <property name="messageConverters">
        <list>
            <ref bean="byteArrayConverter" />
        </list>
    </property>
</bean>
<bean id="byteArrayConverter" class="org.springframework.http.converter.ByteArrayHttpMessageConverter">
</bean>
```
进一步联想，上传文件是不是只能是`post`方法呢？
我们知道，浏览器是不支持RESTful风格中put和delete的
通过测试，使用put提交，直接报415错误，通过源码（org.springframework.web.filter.HiddenHttpMethodFilter）可以看到：
```java
@Override
protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain)
			throws ServletException, IOException {
    HttpServletRequest requestToUse = request;
    if ("POST".equals(request.getMethod()) && request.getAttribute(WebUtils.ERROR_EXCEPTION_ATTRIBUTE) == null) {
	   String paramValue = request.getParameter(this.methodParam);
	   if (StringUtils.hasLength(paramValue)) {
	       requestToUse = new HttpMethodRequestWrapper(request, paramValue);
	   }
    }
    filterChain.doFilter(requestToUse, response);
}
```
从源码可以看出，filter中限制了只能用POST方法提交，那么如果一定要用PUT方法进行上传文件呢？
看了下HiddenHttpMethodFilter注释：
```java
/**
 * {@link javax.servlet.Filter} that converts posted method parameters into HTTP methods,
 * retrievable via {@link HttpServletRequest#getMethod()}. Since browsers currently only
 * support GET and POST, a common technique - used by the Prototype library, for instance -
 * is to use a normal POST with an additional hidden form field ({@code _method})
 * to pass the "real" HTTP method along. This filter reads that parameter and changes
 * the {@link HttpServletRequestWrapper#getMethod()} return value accordingly.
 *
 * <p>The name of the request parameter defaults to {@code _method}, but can be
 * adapted via the {@link #setMethodParam(String) methodParam} property.
 *
 * <p><b>NOTE: This filter needs to run after multipart processing in case of a multipart
 * POST request, due to its inherent need for checking a POST body parameter.</b>
 * So typically, put a Spring {@link org.springframework.web.multipart.support.MultipartFilter}
 * <i>before</i> this HiddenHttpMethodFilter in your {@code web.xml} filter chain.
 *
 * @author Arjen Poutsma
 * @author Juergen Hoeller
 * @since 3.0
 */
```
MultipartFileter：
```java
/**
* Set the bean name of the MultipartResolver to fetch from Spring's
* root application context. Default is "filterMultipartResolver".
*/
public void setMultipartResolverBeanName(String multipartResolverBeanName) {
    this.multipartResolverBeanName = multipartResolverBeanName;
}
```
也就是说，我们可以通过在web.xml中注册一个MultipartFilter，一定要在HiddenHttpMethodFilter之前启动，这样的话就能实现
```xml
<filter>
    <filter-name>MultipartFilter</filter-name>
    <filter-class>org.springframework.web.multipart.support.MultipartFilter</filter-class>
</filter>
<filter-mapping>
    <filter-name>MultipartFilter</filter-name>
    <servlet-name>dispatcher</servlet-name>
</filter-mapping>

<filter>
    <filter-name>HiddenHttpMethodFilter</filter-name>
    <filter-class>org.springframework.web.filter.HiddenHttpMethodFilter</filter-class>
</filter>
```
同时配置：
```xml
<bean id="filterMultipartResolver"
class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
    <property name="maxUploadSize" value="209715200"/>
    <property name="defaultEncoding" value="UTF-8"/>
    <property name="resolveLazily" value="true"/>
</bean>
```
前端：
```javascript
function upload() {
    var form = new FormData(document.getElementById("xx"));
    form.append("_method", 'put');
    $.ajax({
        url: url,
        type: 'post',
        data: form,
        processData: false,
        contentType: false,
        success: function (data) {
            ...
        },
        error: function (e) {
            ...
        }
    });
    ...
}
```
通过java文件上传分析，可以得出，框架为我们做了很多默认配置，为我们省去了很多工作，当然，也有限制，如果我们想挣脱限制，就要找准入口～

参考：
[SpringMVC实现RESTful带有文件上传的put](http://www.cnblogs.com/morethink/p/6378015.html)
