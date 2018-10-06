---
categories:
- 伪技术相关
- golang
tags:
- crawler
- in action
title: 用CookieJar获取一次302/301跳转的Cookie
---

## 问题描述:

在遇到需要登录的场景下,往往需要一次302/301跳转,然而我们的Client会跟着一起跳转,这时如果获取`response.Cookie()`的话,获取到的是跳转后页面的Cookie,会导致登录页面的`Set-Cookie`丢失掉,从而会导致登录失败。

## 解决方案:

在查过一些资料之后,发现Golang引入了类似PY的`CookieJar`,用来存储Client发起请求时产生的Cookies.使用的时候,需要在Client中设置Jar：`Client.Jar=jar`,获取302/301的跳转时产生的Cookie用到CookieJar就很简单了,用`CookieJar.Cookie(Domain)`，比如 http://www.example.com/login 会执行一次302/301跳转,在执行POST操作之后,调用一次
```golang
ParsedURL,_:=url.Parse("http://www.example.com/login")
resCookie:=Client.Jar.Cookies(ParsedURL)
```
那么这里的`resCookie`即是拦截到的302/301跳转时的Cookie.
