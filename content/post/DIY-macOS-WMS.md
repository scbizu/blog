---
title: "DIY MacOS WMS"
date: 2020-06-27T16:47:48+08:00
lastmod: 2020-06-27T16:47:48+08:00
draft: false
keywords: []
description: ""
tags: []
categories: []
author: ""

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

# 调教属于自己的mac窗口管理器(mWMS)

> 一个Geeker就要拥有自己的窗口管理器（强行

## 缘起

自从前年刚入mac生态生态时就在体验各种各样的窗口管理器，但是一直没有找到一款喜欢的。偶然在 @linuxtoy 的一次Retweet里面发现了一个看起来颜值很高的平铺窗口管理器 — [yabai](http://github.com/koekeishiya/yabai)。**颜值高**，**平铺**，**可以自己写代码来调整配置**。这不是我理想的wms原型嘛，于是就在一个周末开始折腾了。

## DIY

> 具体可以参照yabai的[官方wiki](https://github.com/koekeishiya/yabai/wiki)设置

yabai由一份.yabairc驱动，里面定义了所有窗口管理的行为(yabai config)，比如 窗口透明度 ，窗口大小，窗口组织格式（平铺 or 浮动，不止平铺窗口，但是我还是更喜欢平铺XD），把.yabairc设置成可执行并通过`homebrew service`启动之后，yabai就可以在你的mac生效了。

这之后，我们就可以通过yabai提供的命令行工具操作窗口了。比如切换focus的窗口 `yabai -m window —west` , 切换window并转移foucus到这个window `yabai -m window --space 1; yabai -m space --focus 1` ，我当时甚至还写了一个[Alffred Workflow](https://github.com/scbizu/yabai-alfredflow) 来管理窗口。

但是即便有了workflow每次敲命令行也太麻烦了，这时候发现yabai的作者其实还推荐了另一个跟yabai搭配的工具 — [skhd](https://github.com/koekeishiya/skhd) , 这个工具本意是维护一些常用命令的key binding ，但是由于yabai本身就是一个response-less的工具，所以很skhd非常合得来，比如我们可以配置  `cmd + alt + 1` 来mapping到刚才的 `yabai -m window --space 1; yabai -m space --focus 1 (cmd + alt - 1 : yabai -m space --focus 1)，`这时候就可以通过键盘的keybindings来操作窗口了。

作为一个窗口管理工具，没有一个简易的面板来管理的话，颜值其实就没那么高了。因为我们用了yabai之后其实就放弃了mac自带的Dock ，所以一般使用的时候都会 `killall Dock`，这样整个屏幕就看上去太空了，正好可以空出一个塞statusBar的间距。~~于是，yabai会自带一个statusBar，并且statusBar也是可以配置的 (已在最近版本中移除，yabai开始专注窗口管理，不再集成statusbar的功能，把这个扔给用户自己实现了，如果需要的话)。~~

## 变革

变革包括了两个部分:

## 0x01:  MacOS Catalina

可能对于大部分macOS用户来说，Catalina升级是平滑的，beta之后修复了很多bug，macOS继续作为大多数码农的效率工具发光发热，但是，yabai社区却炸开了锅。由于yabai使用的是从macOS Mojave时代起开放的[Apple Scripting Addition](https://forum.latenightsw.com/t/scripting-additions-on-mojave/1550) （这东西Apple的Release Note也没有提到 , 所以， Apple并没有义务对Scripting Addition做兼容，在我升级到Catalina的时候，果然由于yabai的关系，macOS直接白屏退出了，看Crash Log就是Windows Manager Crash导致的。

翻了下issue列表，果然不止我一个人遇到这个问题，找到了这个 [issue #275](https://github.com/koekeishiya/yabai/issues/275) , 并在 [issue #277](https://github.com/koekeishiya/yabai/issues/277)看到了问题的根源：**macOS修改了一个指针地址，导致改变macOS背景透明度的时候，yabai会crash**。 暂时改了yabai窗口透明度(opacity_duration设置为0.0)之后，一切貌似又恢复了正常。

## 0x02: MacOS Catalina 10.15.5

由于我一直没升级yabai的关系(挺稳定的，就一直忘记升级)，在我macOS升级Catalina 10.15.5 这个patch之后，yabai重新启动的时候报了一个错：`payload doesn't support this macOS version` 。没找到相关的issue，但是好在yabai是完全开源的项目，直接看了yabai的代码，发现yabai定义的`OSAX_VERSION` 和 我本机的`OSAX_VERSION` (通过yabai在debug模式下启动的log可以看到)，感觉应该升级yabai到HEAD version可以解决这个问题。先通过brew upgrade更新，发现并没有新版本更新，于是按照[官方的方法](https://github.com/koekeishiya/yabai/wiki/Tips-and-tricks#auto-updating-from-head-via-brew)更新到了HEAD的版本。

再重新install-sa之后，yabai回来了！

但是，咦 ？ 我的statusbar哪里去了 ？难道是新版本移除了statusbar，继续翻issue list，终于找到了[相关的issue](https://github.com/koekeishiya/yabai/issues/486) ，原来是作者不想继续维护这个statusbar，想专心做wms了。issue翻到末尾给了我惊喜，有人单独把原来的statusbar fork出来了一个新的项目，叫做[spacebar](https://github.com/somdoron/spacebar)。看到社区这么活跃，我就更开心了！

## 涅槃

但是，如果还用回原来的statusbar，对于我这种一直想尝试新东西的人就有点索然无味了。抱着试一试的心态，下载了wiki里面提到的mac的widget工具 [Übersicht](http://tracesof.net/uebersicht/) 。这个工具还是挺有趣的，能自己通过`jsx OR coffee script` 编写mac的桌面widget。也就是说，用它就可以DIY自己的statusbar了！

为了配合自定义的statusbar，yabai的新版本开放了窗口padding的设置:

```bash
yabai -m config top_padding                  x
yabai -m config bottom_padding               x
yabai -m config left_padding                 x
yabai -m config right_padding                x
```

用户可以自定义窗口距离屏幕的padding大小来决定自己的statusbar或者一些widget放在哪里。

空出了padding之后，就可以写自己的statusbar了。然而，我根本就写不来 jsx 和 coffee script XD。于是去github上逛了逛，嘛，还是有很多人在用Ubersicht的嘛。找了一个还看得过去的设计就开始自己对着ReactJS的文档XJB改了一个[statusbar](https://github.com/scbizu/nibar)。加windows index tab，加battery icon ，加clock icon等等。

加完之后发现，如果每次依靠Refresh Duration来刷新的话，那我的statusbar还有什么意义，都不能实时反应出我在哪个窗口，其它窗口有没有被占用。

但是，yabai貌似什么都算到了！它开放了一种event notify的方式可以和其它的script进行交互(通过macOS的osascript)，只需要在yabairc里面指定你要交互的app name就可以了，比如：

```bash
yabai -m signal --add event=space_changed \
            action="osascript -e 'tell application id \"tracesOf.Uebersicht\" to refresh widget id \"spaces-primary-jsx\"'"
```

用了几天，感觉确实还行，比之前的statusbar更谐了hhh。

![mac%20mWMS%20e49307fe58264b009fbd2626a7d355d3/Untitled.png](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/9748b096-5af4-4968-bf77-0f52e225e95a/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20200627%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20200627T085102Z&X-Amz-Expires=86400&X-Amz-Signature=5db4e5f95902579db1f2d525d9d95205639c978f38b6f63517b091fba89b14b5&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22)

statusbar交给自己管理之后，确实有很多异想天开的功能可以自己实现：比如实现一个 windows XP 的statusbar也是可以的（如果jsx script写得6的话

## 总结

本篇记录了我在折腾和实际体验yabai时遇到的一些问题和感想。

那么，以上，就酱。

> ~~什么 ？ 你问我敢不敢升级Big Sur，那当然是不敢啊，不过[社区已经有勇士上车了](https://github.com/koekeishiya/yabai/issues/589)，等待后续吧。~~
