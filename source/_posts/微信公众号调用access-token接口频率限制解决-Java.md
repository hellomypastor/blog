title: "微信公众号调用access_token接口频率限制解决(Java)"
date: 2015-05-09 15:02:44
type: "tags"
tags:
- 微信公众平台开发
- Java
---
access_token是公众号的全局唯一票据，公众号调用各接口时都需使用access_token。开发者需要进行妥善保存。access_token的存储至少要保留512个字符空间。access_token的有效期目前为2个小时，需定时刷新，重复获取将导致上次获取的access_token失效。
公众号调用接口并不是无限制的。为了防止公众号的程序错误而引发微信服务器负载异常，默认情况下，每个公众号调用接口都不能超过一定限制，当超过一定限制时，调用对应接口会收到如下错误返回码：
```
{"errcode":45009,"errmsg":"api freq out of limit"}
```
那怎么才能在access_token失效前定时刷新呢？
<!--more-->

## 新建token线程，定时获取token
代码如下：
```java
public class TokenThread implements Runnable 
{
  public static String appid = "";
  public static String appsecret = "";
  public static AccessToken accessToken = null;
  public static String jsapiTicket = null;
  public void run()
  {
    while (true)
    {
      try
      {
      	accessToken = AdvancedUtil.getAccessToken(appid, appsecret);
      	if (null != accessToken)
      	{
      	  jsapiTicket = AdvancedUtil.getJsapiTicket(accessToken.getAccess_token());
      	  if (null != jsapiTicket)
      	  {
      	  	Thread.sleep((accessToken.getExpires_in() - 200) * 1000);
      	  }
      	  else
      	  {
      	  	Thread.sleep(60 * 1000);
      	  }
      	}
      	else
      	{
      	  Thread.sleep(60 * 1000);
      	}
      }
      catch (InterruptedException e)
      {
      	try
      	{
      	  Thread.sleep(60 * 1000);
      	}
      	catch (InterruptedException e1) 
      	{
      	}
      }
    }
  }

}
```

## 初始化servlet
代码如下：
```java
public class InitServlet extends HttpServlet 
{
  private static final long serialVersionUID = 1L;
  public void init() throws ServletException
  {
    TokenThread.appid = Constants.APPID;
    TokenThread.appsecret = Constants.APPSECRET;

    //未配置APPID、APPSECRET时不启动线程
    if ("".equals(TokenThread.appid) || "".equals(TokenThread.appsecret))
    {
    }
    else
    {
      //启动定时获取access_token的线程
      new Thread(new TokenThread()).start();
    }
  }
}
```

## web.xml配置
代码如下：
```xml
<servlet>
  <servlet-name>initServlet</servlet-name>
  <servlet-class>
    com.beibeibang.wechat.servlet.InitServlet
  </servlet-class>
  <load-on-startup>0</load-on-startup>
</servlet>
```

## 调用
代码如下：
```java
String accessToken = TokenThread.accessToken;
```
参考
[获取access token](http://mp.weixin.qq.com/wiki/11/0e4b294685f817b95cbed85ba5e82b8f.html)
[接口频率限制说明](http://mp.weixin.qq.com/wiki/0/2e2239fa5f49388d5b5136ecc8e0e440.html)