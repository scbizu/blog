---
title: 嘛,说说我与Astral的故事吧(第零弹)
categories:
- 伪技术相关
tags:
- Astral
---

# 嘛,说说我与Astral的故事吧(第零弹)

> [GitHub Repo](https://github.com/scbizu/Astral)

## Who is Astral

  Astral是个有趣的名字,这个名字出自《游戏王Zexal》, 是主角游马来自异世界(星光界)的伙伴,又译为“星光体”。取这个名字也是因为自己也想有一个虽然出身或者人格不同,但是能一起并肩作战(中二脸)的伙伴吧。

## Astral as a Wechat bot

  刚开始写Astral的时候是想把他造成一个wechat bot,毕竟现在wechat已经成为了主流IM工具了。用了将近一个下午的时间,造出了这个bot的雏形 和 称为 **饭团** (帮公司的同事决定中午点什么菜)的feature,得益于[wechat-go](http://github.com/songtianyi/wechat-go/)简单易懂的说明和代码封装,就可以很容易地根据demo代码,实现一个自己的简单bot.

## Deploy Astral on server

   在deploy的时候,我遇到了问题。其实,暴力一点的话,可以直接把bin扔到服务器上去跑。但是这样每改一版，就要自己手动deploy一次。

   但是对我这么懒的人来说,这样的方法真是太累了。

  于是,我打开了我尘封已久的DaoCloud账号。添加了GitHub上的Astral代码仓库,DaoCloud上的基于Docker的自动部署之前就玩过,所以一步步很顺利地创建了带Go环境的Container,部署了第一版的Astral。

  具体的Dockerfile可以参考下[我给Astral写的Dockerfile](https://github.com/scbizu/Astral/blob/master/Dockerfile).

## Login your wechat account in the terminal

  然后,问题就又来了。wechat-go提供了两种Scan QR Code的方式。一种打印在Terminal上,即 **TERMINAL_MODE** ,另一种是 **WEB_MODE。**   

   刚开始用的是第一种,毕竟我本地测试一直用得这种,一直测试很方便。但是不知为何DaoCloud的APP日志的Console不支持QR Code。尝试了很多次,未果,遂放弃。

   然后转战第二种实现。也就是WEB_MODE。尝试了几次,发现在Web_MODE下,wechat-go并不会自己创建qrcode所在的文件夹,需要自己创建好文件夹后,才会有qrcode图片。于是,我自己改了下wechat-go,[changes在这](https://github.com/scbizu/wechat-go/commit/441bde85a09735b74bb8d80b1e2a58844f347bc6)(不提PR是因为我加了[dep](https://github.com/golang/dep) 的依赖,这种管理vendor的方法,好像项目的contributor都不太接受,遂放弃PR)。在我实现的这版改动中,启用WEB_MODE之后,会在项目的目录下直接生成qrcode.jpg。用WEB_MODE登录,比TERMINAL_MODE需要多一步:注册文件服务器,让外网可以访问扫码。就像这样:

    go http.ListenAndServe(":8080", http.FileServer(http.Dir("./")))

> 这里不要忘记给Docker的Container暴露端口

   然后,deploy之后,打开 `http(s)://IP(DOMAIN):PORT/qrcode.jpg` 就可以看到二维码图片了,然后扫码就可以登录Wechat了。

## Why Astral is no longer a Wechat bot ?

  过了几个月,发现自己每个月都给自己的VPS续费,然后又不去用服务器资源,简直是太浪费钱了。

  于是,我看了下DaoCloud上Astral的运行情况,发现已经停止了很久了。在Wechat上登录Astral的时候发现,Astral已经是一个无效的账号了。用注册Astral的手机号注册的时候发现变成了一个新号,同时也发现了那些有Astral的微信群中都已经移除掉了Astral,没有任何的退群log就不见了, 应该是Wechat官方直接删了号。

 用新号再次扫码想登录到Web Wechat的时候 发现 Web Wechat已经不支持新号登录了。

  也就是说,Wechat官方应该是放弃维护Web Wechat了。这样看来,是Wechat为了ban掉那些乱发小广告的害群之马,就索性把Wechat Bot的路子给“彻底”封死了。

  那就放弃Wechat吧。让Wechat去死吧！

以上,为我和Astral之间的故事(序篇)。

接下来,我跟Astral之间的故事会围绕着 **用Go去开发一个可轻松扩展并且有趣的Telegram Bot** 为主线去展开,希望之后能跟Astral一起成长:)。
