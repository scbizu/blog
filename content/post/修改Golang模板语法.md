---
categories:
- 伪技术相关
tags:
- golang
- template
title: 修改Golang template语法
---

## 前言

记得之前看过V站(v2ex)的一个讨论帖,内容就是关于前端的模板语法会跟后端的模板语法冲突.我也同意 **统一渲染** 的观点,比如 由后端统一进行data binding 或者由前端来做,但是在实际的开发中,我也还是想着要偷一下懒的,所以哪端方便就由哪端来做,也不会去刻意地进行MVVM的划分。在这种情况下 就很容易出现服务端的data binding 跟 前端的出现冲突。

这种现象 随着前端越来越火 也会有加剧的趋势,特别是引入了NodeJS的技术栈之后,前端的职责更多了,相反的,后端现在已经不需要怎么关心数据渲染的事情了,只要专注于数据结构,专注于服务的性能,专注于API的高度解耦就可以了。但是 也并不是说 后端失去了Render页面的能力,golang 的 `template`包很强大,同样也是数据驱动。在一些后端直接Render比较方便的场合 再把API给前端 然后前端再进行一次`fetch()` 之后序列化数据再渲染,不是有点绕了？

## 说明

由于最近想玩玩大热的Vue,顺便支持下国产,于是在公司的项目里使用了Vue(由于前端的量比较少,所以就任性了一把)。Vue也是数据驱动(JSON),跟`template`差不多,所以很快就上手了。

但是用着用着 就开始跟`Go`的`template`打架了。因为都是`{ {  } }`这种标记 所以在Golang template的`Pre-compile`中 由于解析到了 Vue的模板变量 所以就会报not defined的错误。(由于懒)  就想着把golang的`template`的语法标记换掉,接触过Beego的都知道 在beego里可以通过 `beego.TemplateLeft`和`beego.TemplateRight`来改变标记,但是在没有用到beego的环境下,要怎么更改呢 ?  在template包下有个`Delims`函数 我们要做的就是 **在compile模板(parse)之前设置好我们想变成的标记语言** 。可以这样写:

```
temp := template.New("static/*.html").Delims("<<<", ">>>")
resTemp := template.Must(temp.ParseGlob("static/*.html"))
```

这之后  就把`{ {  } }`标记 替换成了 `<<<>>>`.

## 想法

这种方法非常不适合出现在有大量前端页面的场合.因为本着前后端分离的原则进行设计的话 前端必须给他api fetch到页面上的所有数据，再由前端进行data binding，这样不管是对于`前端的组件化`还是`后期维护性(耦合性)`来说都是有好处的。

但是 如果站在全栈工程师的角度来看待这个问题的话 我觉得混在一起写也不是不可以嘛,或者在自己side project里面去这么写也是省时省力的好方法。(我这里只是揣测了一下全栈大牛们的想法 。

以上。。。。
