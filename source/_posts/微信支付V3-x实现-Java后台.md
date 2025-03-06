title: "微信支付V3.x后台实现(Java)"
date: 2015-05-09 13:25:50
type: "tags"
tags:
- 微信公众平台开发
- Java
- SpringMVC
---
最近做微信公众号开发，用到了`网页中调起微信支付`的功能，微信公众平台提供的DEMO是基于PHP的，没有Java的，到网上查阅资料也都是`V2.x`的，而现在微信支付已经更新到了`V3.x`版本，经过几天的探索，终于成功实现，将实现过程总结下。
<!--more-->
## 获取签名证书，初始化JS-API
在网页中JS调起微信支付需用到微信JS-SDK，首先得初始化JS-API，后台将`签名证书`传给前端。
- 第一步，获取jsapiTicket
```java
//获取AccessToken
public static AccessToken getAccessToken(String appId, String appSecret) 
{
  AccessToken accessToken = null;
  String requestUrl = "https://api.weixin.qq.com/cgi-bin/token?grant_type=client_credential&appid=APPID&secret=APPSECRET";
  requestUrl = requestUrl.replace("APPID", appId);
  requestUrl = requestUrl.replace("APPSECRET", appSecret);
  String result = CommonUtil.httpRequest(requestUrl);
  JSONObject jsonObject = JSONObject.fromObject(result);
  if (null != jsonObject) 
  {
    try 
    {
      accessToken = new AccessToken();
      accessToken.setAccess_token(jsonObject.getString("access_token"));
      accessToken.setExpires_in(jsonObject.getInt("expires_in"));
    } 
    catch (Exception e) 
    {
      accessToken = null;
      int errorCode = jsonObject.getInt("errcode");
      String errorMsg = jsonObject.getString("errmsg");
    }
  }
  return accessToken;
}
//获取jsapiTicket
public static String getJsapiTicket(String accessToken) 
{
  String jsapiTicket = null;
  String requestUrl = "https://api.weixin.qq.com/cgi-bin/ticket/getticket?access_token=ACCESS_TOKEN&type=jsapi";
  requestUrl = requestUrl.replace("ACCESS_TOKEN", accessToken);
  String result = CommonUtil.httpRequest(requestUrl);
  JSONObject jsonObject = JSONObject.fromObject(result);
  if (null != jsonObject) 
  {
    try 
    {
      jsapiTicket = jsonObject.getString("ticket");
    }
    catch (Exception e)
    {
      int errorCode = jsonObject.getInt("errcode");
      String errorMsg = jsonObject.getString("errmsg");
    }
  }
  return jsapiTicket;
}
```
`access_token和jsapiTicket需控制调用频率`，参考[微信公众号调用access_token接口频率限制解决(Java)](http://hellomypastor.net/2015/05/09/%E5%BE%AE%E4%BF%A1%E5%85%AC%E4%BC%97%E5%8F%B7%E8%B0%83%E7%94%A8access-token%E6%8E%A5%E5%8F%A3%E9%A2%91%E7%8E%87%E9%99%90%E5%88%B6%E8%A7%A3%E5%86%B3-Java/)
- 第二步，获取sign签名证书
```java
public static Map<String, String> sign(String jsapi_ticket, String url) 
{
  Map<String, String> ret = new HashMap<String, String>();
  String nonce_str = create_nonce_str();
  String timestamp = create_timestamp();
  String string1;
  String signature = "";
  // 注意这里参数名必须全部小写，且必须有序
  string1 = "jsapi_ticket=" + jsapi_ticket + "&noncestr=" + nonce_str + "&timestamp=" + timestamp + "&url=" + url;
  try 
  {
    MessageDigest crypt = MessageDigest.getInstance("SHA-1");
    crypt.reset();
    crypt.update(string1.getBytes("UTF-8"));
    signature = byteToHex(crypt.digest());
  } 
  catch (NoSuchAlgorithmException e) 
  {
    return null;
  } 
  catch (UnsupportedEncodingException e) 
  {
    return null;
  }
  ret.put("jsapi_ticket", jsapi_ticket);
  ret.put("nonceStr", nonce_str);
  ret.put("timestamp", timestamp);
  ret.put("signature", signature);
  return ret;
}
```
## 获取微信支付配置参数
微信支付参数需要的有3个:a.微信分配的公众账号ID（appid）;b.微信支付分配的商户号（mch_id/PartnerId）；c.商户密钥（PartnerKey）
## 调用统一下单接口，生成预支付订单号
代码如下：
```java
/**
* 生成预支付订单
* 
* @return
*/
public String submitXmlGetPrepayId() 
{
  // 创建HttpClientBuilder
  HttpClientBuilder httpClientBuilder = HttpClientBuilder.create();
  HttpPost httpPost = new HttpPost(unifiedorder);
  String xml = getPackage();
  StringEntity entity;
  String result = null;
  try 
  {
    entity = new StringEntity(xml, "utf-8");
    httpPost.setEntity(entity);
    HttpResponse httpResponse;
    httpResponse = closeableHttpClient.execute(httpPost);
    HttpEntity httpEntity = httpResponse.getEntity();
    if (httpEntity != null)
    {
      result = result.replaceAll("<![CDATA[|]]>", "");
      String prepay_id = Jsoup.parse(result).select("prepay_id").html();
      this.prepay_id = prepay_id;
      if (prepay_id != null)
      {
        return prepay_id;
      }
    }
    closeableHttpClient.close();
  } 
  catch (Exception e) 
  {
    e.printStackTrace();
  }
  return prepay_id;
}
```
## 生成带支付签名的订单凭据并返回
代码如下：
```java
public class WXPay 
{
  public static String createPackageValue(String appid, String appKey, String prepay_id)  
  {
    SortedMap<String, String> nativeObj = new TreeMap<String, String>();
    nativeObj.put("appId", appid);
    nativeObj.put("timeStamp", OrderUtil.GetTimestamp());
    Random random = new Random();
    String randomStr = MD5.GetMD5String(String.valueOf(random.nextInt(10000)));
    nativeObj.put("nonceStr", MD5Util.MD5Encode(randomStr, "utf-8").toLowerCase());
    nativeObj.put("package", "prepay_id=" +prepay_id);
    nativeObj.put("signType", "MD5");
    nativeObj.put("paySign", createSign(nativeObj, appKey));
    System.out.println(JSONObject.fromObject(nativeObj).toString());
    return JSONObject.fromObject(nativeObj).toString();
  }
  /**
   * 创建md5摘要,规则是:按参数名称a-z排序,遇到空值的参数不参加签名。
   */
  @SuppressWarnings("rawtypes")
  public static String createSign(SortedMap<String, String> packageParams, String AppKey) 
  {
    StringBuffer sb = new StringBuffer();
    Set es = packageParams.entrySet();
    Iterator it = es.iterator();
    while (it.hasNext()) 
    {
      Map.Entry entry = (Map.Entry) it.next();
      String k = (String) entry.getKey();
      String v = (String) entry.getValue();
      if (null != v && !"".equals(v) && !"sign".equals(k) && !"key".equals(k)) 
      {
        sb.append(k + "=" + v + "&");
      }
    }
    sb.append("key=" + AppKey);
    String sign = MD5Util.MD5Encode(sb.toString(), "UTF-8").toUpperCase();
    return sign;
  }
}
```
## 调起微信支付（发起微信支付请求）
## 用户确认金额并输入支付密码
## 向用户展示支付结果
参考
[JS-SDK-API](http://mp.weixin.qq.com/wiki/7/aaa137b55fb2e0456bf8dd9148dd613f.html)
[微信支付PDF文档](https://mp.weixin.qq.com/paymch/readtemplate?t=mp/business/course3_tmpl&lang=zh_CN)
[这篇文章](http://blog.csdn.net/omsvip/article/details/43342957)。
