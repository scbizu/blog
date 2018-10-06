---
categories:
- 伪技术相关
- ssh
tags:
- 当XX 发生了什么?
- theory
title: 当我们ssh {{user}}@{{ip}} 发生了什么?
---

当我们需要登录我们的云服务器时,经常会用到ssh命令,但是当我们`ssh root@192.168.1.1` 时到底发生了什么?

> 基于ssh-1协议的描述

![ssh_flow](http://7xqdui.com1.z0.glb.clouddn.com/ssh_flow.png)
