title: "Flask+SAE搭建微信后台初探"
date: 2015-05-11 23:54:24
type: "tags"
tags:
- Flask
- Python
- 微信公众平台开发
---
Flask+SAE搭建微信后台初探
<!--more-->
## 申请微信公众号
申请通过之后，将模式改为`开发者模式`，并填入自己服务器的`url`及`token`（令牌）。
## 在SAE上新建Python项目
- 通过SVN管理版本
- Flask框架
具体见[SAE入门指南](http://sae.sina.com.cn/doc/python/tutorial.html)

## 微信公众号与SAE应用的对接和验证
处理流程为：
- 用户发送请求到微信服务器
- 微信服务器发送请求到SAE
- SAE返回消息给微信服务器
- 微信服务器返回消息给用户

SAE上处理Http请求，验证之后返回，代码如下：
```python
# -*- coding:utf8 -*-
import time
from flask import Flask, g, request, make_response
import hashlib

@app.route('/', methods = ['GET', 'POST'] )
def wechat_auth():
  if request.method == 'GET':
    token = 'xxxxxxxxxxx' # your token
    query = request.args  # GET 方法附上的参数
    signature = query.get('signature', '')
    timestamp = query.get('timestamp', '')
    nonce = query.get('nonce', '')
    echostr = query.get('echostr', '')
    s = [timestamp, nonce, token]
    s.sort()
    s = ''.join(s)
    if ( hashlib.sha1(s).hexdigest() == signature ):
      return make_response(echostr)
```
按照SAE中Python及Flask规范将代码提交到SVN中，将应用地址和token填入微信公众平台进行验证，如果设置成功，即完成验证。
### 处理用户消息（以文本消息为例）
当普通微信用户向公众账号发消息时，微信服务器将POST消息的XML数据包到开发者填写的URL上，文本消息格式如下：
```
<xml>
 <ToUserName><![CDATA[toUser]]></ToUserName>
 <FromUserName><![CDATA[fromUser]]></FromUserName> 
 <CreateTime>1348831860</CreateTime>
 <MsgType><![CDATA[text]]></MsgType>
 <Content><![CDATA[this is a test]]></Content>
 <MsgId>1234567890123456</MsgId>
</xml>
```
我们将此消息解析出来，然后再构造xml返回给用户，`ToUserName和FromUserName`需对换，代码如下：
```python
# Get the infomations from the recv_xml.  
xml_recv = ET.fromstring(request.data)
ToUserName = xml_recv.find("ToUserName").text
FromUserName = xml_recv.find("FromUserName").text
Content = xml_recv.find("Content").text 
# 此处可处理Content
reply = "<xml><ToUserName><![CDATA[%s]]></ToUserName><FromUserName><![CDATA[%s]]></FromUserName><CreateTime>%s</CreateTime><MsgType><![CDATA[text]]></MsgType><Content><![CDATA[%s]]></Content><FuncFlag>0</FuncFlag></xml>"
response = make_response( reply % (FromUserName, ToUserName, str(int(time.time())), Content ) )
response.content_type = 'application/xml'
return response 
```
到此，在微信公众号里回复消息，应该就会有消息回复啦~

参考
[SAE入门指南之Python](http://sae.sina.com.cn/doc/python/tutorial.html)
[微信接入指南](http://mp.weixin.qq.com/wiki/17/2d4265491f12608cd170a95559800f2d.html)
