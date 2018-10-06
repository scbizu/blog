---
categories:
- 伪技术相关
title: 那么就开始使用Context吧
date: 2017-06-25 16:49:56
tags:
- in action
- golang
thumbnail: https://ws1.sinaimg.cn/large/005uxZXHgy1fgxk3uf5wbj30hs071dia.jpg
---

> Context并不是银弹,使用时需注意其使用场景。

## Context

golang的`context`包的"前身"来自`golang.org/x/net/context`,在Go 1.7时被引入到标准库,现为`context`,使用之前`golang.org/x/net/context`建议切到`context`开发。包如其名,这个包主要实现了"上下文"的相关功能:包括了代码调用栈,传递scope等。`context`在开发API和RPC接口时发挥了巨大的作用。

在微服务流行的今天,各个服务互相独立,所以,在调用时能有效地传递当前上下文变得尤为重要(通常是一些用户鉴权,请求的deadline等等).
