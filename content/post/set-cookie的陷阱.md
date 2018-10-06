---
categories:
- Crawler
tags:
- Solution
title: 关于set-cookie的注意事项
---

## Set-Cookie

关于Set-Cookie的具体说明可以在[MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie)(参见RFC 6265 section 4.1)上很容易翻到,这边就不细说了。简单说下,就是浏览器(B端)发起请求(Request),接到服务器端(S端)的响应(Response)的时候会检查Response的Header中的Set-Cookie字段,如果存在,那么浏览器就会把这个字段加到Request的Cookie中去。

## 注意点

在爬虫的实践中,很多时候都需要获取到页面的Cookie,来高效地模仿浏览器行为。对于没有`Set-Cookie`的页面 可能一次Request就可以拿到这个页面的Cookie,之后的模拟登录操作也会手到擒来。但是,很多人会有疑惑,为什么登陆的操作会没有成功?这时不妨静下心来,仔细分析下报文,你会发现这时你Request Header的Cookie字段 跟 正常浏览时对应不起来。看了下Resonse Header又会发现多出了`Set-Cookie`字段 **这个字段就是服务器返回给你的Cookie ,是可以通过`Response.Cookies()`获取到的。**

既然知道了这个是Server给我们的应答Cookie ,接下来就比较好办了。我们需要做的就是模仿浏览器,重新发起一次带着这些Cookie的Request给服务器就可以了。  

所以,看到Response Header里面有`Set-Cookie`字段时,我们需要发起两次请求.通用爬虫可以跟浏览器一样,**第一次Request检查Response Header里的Set-Cookie字段,第二次Request再带上Server 返回的Cookie去进行模拟登录等操作。**
