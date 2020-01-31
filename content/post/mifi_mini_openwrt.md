---
title: "mifi mini with openwrt"
date: 2020-01-31T16:38:11+08:00
lastmod: 2020-01-31T16:38:11+08:00
draft: false
keywords: ["openwrt"]
description: ""
tags: ["mifi","openwrt"]
categories: ["翻车现场","XJB折腾"]
author: "scnace"

# You can also close(false) or open(true) something for this content.
# P.S. comment can only be closed
comment: false
toc: false
autoCollapseToc: false
postMetaInFooter: false
hiddenFromHomePage: false
# You can also define another contentCopyright. e.g. contentCopyright: "This is another copyright."
contentCopyright: false
reward: false
mathjax: false
mathjaxEnableSingleDollar: false
mathjaxEnableAutoNumber: false

# You unlisted posts you might want not want the header or footer to show
hideHeaderAndFooter: false

# You can enable or disable out-of-date content warning for individual post.
# Comment this out to use the global config.
#enableOutdatedInfoWarning: false

flowchartDiagrams:
  enable: false
  options: ""

sequenceDiagrams:
  enable: false
  options: ""

---

# mifi mini的openwrt之旅

> 本文是劝读者千万不要想不开再去给小米wifi迷你 刷 openwrt 的最新版本了  ！！

## miboy重新归来

之前家里的书房wifi虽说用的就是我毕业的时候从学校带回来的  miwifi  mini  ，但是，固件一直用的很老版本的Pandorabox  , 大概是基于openwrt 14.0x 改的那种吧，然后一直在这上面挂着个梯子用来路由科学上网，但是并不好用，路由间歇性崩溃，甚至都影响到正常的上网了。这次正好趁着过年回来就想着升级下固件，直接换成openwrt的最新版，再上个Clash，满血复活，不是爽的一笔 ？

## wdnmd

 所以，第一步就肯定是重刷固件了 ，找了好久才找到Pandorabox的刷固件入口(有点瞎)，上传官网下载的openwrt 19.07，然后，默默地等待奇迹发生。

蓝灯亮起，从闪烁变成常亮了，openwrt固件刷入成功，那就一起来看看发生了啥吧。诶？貌似没有发现默认的open wifi ？没有错， **openwrt在首次刷入时只能从路由器的wan口拉网线出来**。

这都是些小事，接好网线输入192.168.1.1之后进入了openwrt 19.07的 LuCI页面，接着开始配置wifi试试，卧槽？这个AC mode的5G wifi为毛不能选择自动宽度，只能手选20/40/60 ?  试了下，mac在 40/60  的宽度下不能连接上这个 wifi，只能选择20 。（苹果官网的路由器配置里面倒是建议选择自动设置宽度的）

不知道是不是这个设置的原因，5G wifi的信号隔两堵墙就完全搜不到了( 也是指我的mac)，然后我只能在卧室的时候使用2.4G的wifi了。

这也不是多大的事，这次刷机的首要需求是给路由器安上Clash，因为我本身在mac上也用了大半年的 Clash了，感觉很不错。如果能在路由上就进行代理，那效果也应该不错。于是，按照 luci-app-clash的安装说明开始安装，第一次 opkg  install 的时候失败了，提示依赖没有满足，因为我记得 opkg会自动向feed里面的src拉取对应的包，所以抱着疑问查看了 opkg的 feed list，看来是因为墙的关系导致pull失败了，于是换了几个国内的镜像站(最后用了清华大学的LEDE)，有一件事挺搞笑的 ，我一直以为miwifi mini是  x86_64 架构的 ，无脑换了x86_64的源之后各种报错，`uname -r`之后才发现是mips的，更搞的是看cpu proc info竟然是`mips_24kc`的 ，瞬间就觉得这个路由器应该留着，以后就是古董了XD。

## mips_24kc

真正劝退我的，不是别的，就是这个`mips_24kc`的奇葩架构。

在[luci-app-clash](https://github.com/frainzy1477/luci-app-clash/releases)装完之后(这其实也有坑，貌似是openwrt19.07的 LuCI 把一些lua的包放到自己的代码库去了，所以有些以前版本的Lua脚本会不适用，详见[这个issue](https://github.com/frainzy1477/luci-app-clash/issues/140))，就轮到要装Clash的Client了。在[提供的clash release](https://github.com/frainzy1477/clash_dev/releases)里面没有明确的适合`mips_24kc`的版本，只能自己碰运气尝试mipsle和mips的版本，这里还要区分softfloat和hardfloat的版本([什么是softfloat/hardfloat](https://www.raspberrypi.org/forums/viewtopic.php?t=11177))，在骂了无数次fxxk之后，找了一个勉强可以跑起来的版本mips的softfloat的版本，也有不少人在讨论这个事，[比如这个](https://github.com/antonchen/clash-for-openwrt/issues/1)，[还有这个](https://github.com/Dreamacro/clash/issues/192)。虽然跑起来了，但是跑了没有10分钟就因为内存不足OOM了。出于好奇，查了下V2ray对于`mips_24kc`的支持，果然有[相关issue](https://github.com/v2ray/v2ray-core/issues/396)在讨论这个事，因为mini是[可以开启FPU模拟的](https://wiki.debian.org/InstallingDebianOn/Xiaomi/MiWifi_mini)，貌似理论上是可以装上v2ray的，但是，你指望它 tls/ws decrypt/decode 就有点为难它了，可以尝试下最简单的加密倒是有理论可行性的。（不过倒也可以试试用tinygo去编译下Clash试试，也许会有新的发现

即便是Go本身对mips的支持，还是[有很多坑的](https://github.com/golang/go/issues?utf8=%E2%9C%93&q=mips)。

呀，话题扯远了，在经历了好几次OOM之后，我放弃Clash(v2ray)了，卸掉luci-app-clash之后，iptable rule貌似也被污染了，内网到外网的请求都被iptable drop掉了，终于，我按下了路由器的reset键刷了老毛子。

## 总结

如果下次我有x86的路由器了，我再来折腾openwrt吧。

但是，有一点写在最后，openwrt社区还是很厉害的，即便是19.07最新版，对低端路由器的兼容做得还是很好，至少裸固件是可以跑起来的，而且是CPU占用很低！
