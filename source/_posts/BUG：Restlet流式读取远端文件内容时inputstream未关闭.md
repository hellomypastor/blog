title: BUG：Restlet流式读取远端文件内容时inputstream未关闭
date: 2017-08-01 00:25:10
type: "tags"
tags:
- Java
categories: 疑难杂症
---
利用Restlet流式读取远端文件时，如果没有手动将inputstream关闭，那么InputRepresentation会自动将inputstream关闭吗？
<!--more-->
Restlet 服务端接收rest请求后，读取文件，将文件转为流，代码如下：
```java
public class XXResource extends ServerResource
{
    @Get
    public InputRepresentation readFile() throws FileNotFoundException
    {
        File file = new File("/root/test.txt" );
        InputStream inputStream = new FileInputStream(file);
        InputRepresentation inputRepresentation = new InputRepresentation(
                inputStream);
        return inputRepresentation;
    }
}
```
我们都知道，将文件转为流，在使用结束后需要将流关闭，否则IO资源就会被一直占着，其他方法没法使用，造成资源浪费
进一步考虑，如果此处不需要关闭流，那么在InputRePresentation中应该会把流释放，那么有没有释放呢，我们看InputRepresentation源码：
```java
public void release()
{
    if(this.stream != null)
    {
        try
        {
            this.stream.close();
        } catch (IOException var2) 
        {
            Context.getCurrentLogger().log(Level.WARNING, "Error while releasing the representation.", var2);
        }
        this.stream = null;
    }
    super.release();
}
```
可以看到，此方法为public公用方法，没有地方调用，难道需要手动调用InputRePresentation的release()方法？
看了下StackOverFlow上这个[问答](https://stackoverflow.com/questions/22658056/restlet-not-closing-inputrepresentation-streams)，说的是这个确实是个问题，更新了新版本之后还是没有看到在哪儿调用release()方法，结果回答的人又让手动调用release()方法，这。。

看了下这篇[讨论](http://restlet-discuss.1400322.n2.nabble.com/PROBLEM-WITH-org-restlet-representation-InputRepresentation-IN-1-2-M2-td3055389.html)，明天换个新版本试试～

##continue
今天下载了restlet2.3.9的源码（Maven工程）：
```xml
<repositories>
    <repository>
        <id>maven-restlet</id>
        <name>Public online Restlet repository</name>
        <url>http://maven.restlet.org</url>
    </repository>
</repositories>
<dependencies>
	<dependency>
        <groupId>org.restlet.jee</groupId>
        <artifactId>org.restlet</artifactId>
        <version>2.3.9</version>
    </dependency>
</dependencies>
```
通过从InputRePresentation逐层往父类查找，在RepresentationInfo中找到release()方法的解释：
```java
/**
 * Releases the representation and all associated objects like streams,
 * channels or files which are used to produce its content, transient or
 * not. This method must be systematically called when the representation is
 * no longer intended to be used. The framework automatically calls back
 * this method via its connectors on the server-side when sending responses
 * with an entity and on the client-side when sending a request with an
 * entity. By default, it calls the {@link #setAvailable(boolean)} method
 * with "false" as a value.<br>
 * <br>
 * Note that for transient socket-bound representations, calling this method
 * after consuming the whole content shouldn't prevent the reuse of
 * underlying socket via persistent connections for example. However, if the
 * content hasn't been read, or has been partially read, the impact should
 * be to discard the remaining content and to close the underlying
 * connections.<br>
 * <br>
 * Therefore, if you are not interested in the content, or in the remaining
 * content, you should first call the {@link #exhaust()} method or if this
 * could be too costly, you should instead explicitly abort the parent
 * request and the underlying connections using the {@link Request#abort()}
 * method or a shortcut one like
 * {@link org.restlet.resource.ServerResource#abort()} or
 * {@link Response#abort()}.
 */
 public void release() {
    setAvailable(false);
 }
```
查看此方法被引用的地方，其中最重要的一处为ServerCall类的sendResponse方法：
```java
/**
 * Sends the response back to the client. Commits the status, headers and
 * optional entity and send them over the network. The default
 * implementation only writes the response entity on the response stream or
 * channel. Subclasses will probably also copy the response headers and
 * status.
 * 
 * @param response
 *            The high-level response.
 * @throws IOException
 *             if the Response could not be written to the network.
 */
 public void sendResponse(Response response) throws IOException {
    if (response != null) {
        // Get the connector service to callback
        Representation responseEntity = response.getEntity();
        ConnectorService connectorService = ConnectorHelper
                    .getConnectorService();

        if (connectorService != null) {
            connectorService.beforeSend(responseEntity);
        }
        OutputStream responseEntityStream = null;
        try {
            writeResponseHead(response);
            if (responseEntity != null) {
                responseEntityStream = getResponseEntityStream();
                writeResponseBody(responseEntity, responseEntityStream);
            }
        } finally {
            if (responseEntityStream != null) {
                try {
                    responseEntityStream.flush();
                    responseEntityStream.close();
                } catch (IOException ioe) {
                    // The stream was probably already closed by the
                    // connector. Probably OK, low message priority.
                    getLogger().log(Level.FINE,
                                        "Exception while flushing and closing the entity stream.",
                                        ioe);
                }
            }
            if (responseEntity != null) {
                responseEntity.release();
            }
            if (connectorService != null) {
                connectorService.afterSend(responseEntity);
            }
        }
    }
 }
```
##拨开云雾见天日 守得云开见月明
通过分析，在服务端返回消息给客户端时，会将Representation进行release()，那么InputStream也会释放～

进一步联想：无论是struts／spring或者其他框架，其实都是基于servlet，那么servlet中的流是如何关闭的呢？

有人说：在Servlet中，因为filter的关系，很多时候需要把response里面的内容拿出来，进行过滤，比如编码上的问题，如果你在自己的response里面getWriter().close()掉，其他的filter会出现异常。Servlet最后会帮你关闭的，放心。如果自己用PrintWriter的话，还是得关闭。

有人说：servlet生命周期，用户可以在doGet或doPost中自己关闭输出流；也可以在destory中关闭释放；如果还没有做操作，destory，后会释放servlet实例，自然释放了servlet占用的所有资源。

其实，一个filter、servlet只有一个实例来处理所有请求，`最基础的知识`,`servlet实例中压根没有request、response`，destory想释放估计心有余而力不足，也不要指望在destory里关闭response，他压根不是干这个用的

查看tomcat的源码可以发现，resoponse如果没有关闭，tomcat会自动关闭，org.apache.catalina.core.ApplicationDispatcher.doForward(ServletRequest request, ServletResponse response)方法，最后一段代码是：
```java
// Close anyway
try {
    PrintWriter writer = response.getWriter();
    writer.close();
} catch (IllegalStateException e) {
    try {
        ServletOutputStream stream = response.getOutputStream();
        stream.close();
    } catch (IllegalStateException f) {
        // Ignore
    } catch (IOException f) {
        // Ignore
    }
} catch (IOException e) {
    // Ignore
}
```
可以清楚看到不管你有没有关闭，tomcat都重新关闭了一次

所以说，javax.servlet是一种标准，是给人实现的。

response应该是由Connector类实现的。对外是(javax.servlet.response) response 把方法限定在javax.servlet.response范围。

所以你想要看实现。
我是看的是tomcat4.1.2(深入剖析tomcat这本书是用这个的。)
org.apache.catalina.connector.http.HttpProcessor  response.finishResponse();

好了，就到此为止了，通过此次分析颇有感触，要真正弄懂底层原理才能更好的运用自如！