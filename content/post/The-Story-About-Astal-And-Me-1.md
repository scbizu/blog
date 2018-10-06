---
title: 嘛,说说我与Astral的故事吧(第一弹)
categories:
- 伪技术相关
tags:
- Astral
---
# 嘛,说说我和Astral的故事吧(第一弹)

## From Wechat bot to Telegram bot

在序篇中,我曾提到写Astral的初衷是个Wechat Bot,所以,在引入了wechat-go的依赖之后,这次转移必须要在尽量少改代码的情况下完成。

因为wechat-go"**把所有新的feature都当做plugin**"的设计模式,使得这次转变异常轻松,这也是我在序中说wechat-go代码清晰的主要原因。接下来,我要做的就没有想象的那么复杂了,甚至文件目录都不用变。只要把底层的包从wechat的sdk换成Telegram的sdk就可以了。

然而,为了祭奠自己之前死去的脑细胞,我还是在Astral中留下了wechat-go.

换句话说,在astral-plugin下的**逻辑**都是可以被复用的。

## Combine telegrambotapi with wechat-go

在GitHub找了一圈Telegram SDK的Go实现,最后还是选了[@Syfaro](https://github.com/Syfaro)的[telegram-bot-api](https://github.com/go-telegram-bot-api/telegram-bot-api),并且加入了Telegram bot API group。

tgbotapi实现了大多数Telegram Bot的大多数功能,但是从提供的demo看,貌似需要把Register和feature的逻辑写在一块。所以,我开始着手抽象tgbotapi的实现逻辑,让其可以达到和wechat-go一样非常方便扩展的效果。

以代码为例,可以看到demo中的代码是酱紫的:

    updates := bot.ListenForWebhook("/" + bot.Token)

    	for update := range updates {
    		log.Printf("%+v\n", update)
    	}

作为demo,这样没啥问题,但是如果所有feature写在`range updates` 的for循环里面,这样代码的扩展性会非常糟糕,并且代码结构也不够优雅。

参照着wechat-go的实现 以及 一点自己的想法,最后这边代码会变成这个样子:

    for update := range updatesMsgChannel {
    		log.Printf("[raw msg]:%+v\n", update)

    		if update.Message == nil {
    			continue
    		}
    		var msg tgbotapi.MessageConfig
    		msg = plugin.RegistTGEnabledPlugins(update.Message)
    		//default msg
    		if msg.Text == "" || msg.ChatID == 0 {
    			msg = tgbotapi.NewMessage(update.Message.Chat.ID, update.Message.Text)
    		}
    		msg.ReplyToMessageID = update.Message.MessageID
    		bot.Send(msg)
    	}

这样会让代码清晰很多,并且这边的实现在之后迭代新的feature时也不用改了。把改动限制在增加代码,而非改动代码上了。

## Use telegrambotapi

Telegram Server 和 部署Bot的Server 的通讯是基于HTTPS,所以需要给自己的VPS上HTTPS证书。

- Setup HTTPS Server

  给自己的服务器添加HTTPS证书,可以参照着[Lets encrypt的certbot设置教程](https://certbot.eff.org/)一步一步下去。

  对于Telegram Bot,也可以使用自签证书。

      openssl req -x509 -newkey rsa:2048 -keyout key.pem -out cert.pem -days 3560 -subj "//O=Org\CN=Test" -nodes

- Reverse Proxy (nginx)

  给自己的服务器设置好HTTPS证书之后,就可以保证Telegram Server和自己的Server通讯正常了。

  然而Astral的Listen Server是HTTP的,所以,需要把Telegram Server发来的请求代理到Astral Listen Port,只需要给`nginx.conf` 加上一条反代规则:

      			location /{
      					#PORT是telegram bot监听的端口
               proxy_pass http://127.0.0.1:PORT;
             }

  这样Astral就可以顺利接收到Telegram的请求了。

- Test Your Bot

  对telegrambotapi的demo稍加改造,我们就可以得到一个基于`webhook`的一个可运行版本:

      //ListenWebHook is the tg api webhook mode
      func ListenWebHook(debug bool) (err error) {
      	bot, err := connectTG()
      	if err != nil {
      		return
      	}
      	if debug {
      		bot.Debug = true
      		log.Printf("bot auth passed as %s", bot.Self.UserName)
      	}
      	bot.RemoveWebhook()
      	cert := getcert.NewDomainCert(tgAPIDomain)
      	domainWithToken := fmt.Sprintf("%s%s", cert.GetDomain(), token)
      	if _, err = bot.SetWebhook(tgbotapi.NewWebhook(domainWithToken)); err != nil {
      		log.Printf("notify webhook failed:%s", err.Error())
      		return
      	}

      	info, err := bot.GetWebhookInfo()
      	if err != nil {
      		log.Panicln(err)
      	}
      	log.Println(info.LastErrorMessage, info.LastErrorDate)

      	pattern := fmt.Sprintf("/%s", token)
      	updatesMsgChannel := bot.ListenForWebhook(pattern)
      	log.Printf("msg in channel:%d", len(updatesMsgChannel))

      	port := fmt.Sprintf(":%s", os.Getenv("LISTENPORT"))

      	go http.ListenAndServe(port, nil)

      	for update := range updatesMsgChannel {
      		log.Printf("[raw msg]:%+v\n", update)

      		if update.Message == nil {
      			continue
      		}
      		var msg tgbotapi.MessageConfig

      		if msg.Text == "" || msg.ChatID == 0 {
      			msg = tgbotapi.NewMessage(update.Message.Chat.ID, update.Message.Text)
      		}
      		msg.ReplyToMessageID = update.Message.MessageID
      		bot.Send(msg)
      	}

      	return
      }

  这边值得一提的是,Telegram的bot token需要跟@botfather PY一下,才可以获得。

  然后,代码中的`domainWithToken`就是你的域名或者IP(也就是上一步设置的反代路径)后加上申请得到的`Token`.

- Don't Have Own VPS

  理论上使用`long pulling`的方式部署在本地应该也可以。即用一个goroutine隔一段时间去向TG Server请求message。如果checkpoint设置短的话,也可以达到`webhook`的效果。

## Conclusion

  这一章主要介绍了 **一些tgbotapi的使用技巧** 和 **在VPS上配置TG bot的相关信息** 。

  以上,应该就可以让Bot demo运行起来了。

下一章,会主要侧重一个具体的plugin来讨论。  

## Reference

- [Telegram Bot Doc](https://core.telegram.org/bots/api#getting-updates)
- [Go TG Bot API](https://github.com/go-telegram-bot-api/telegram-bot-api/blob/master/README.md)
- [Python Telegram Bot Wiki](https://github.com/python-telegram-bot/python-telegram-bot/wiki/Webhooks)
