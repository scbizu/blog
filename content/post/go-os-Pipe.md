---
title: "Go os.Pipe()"
date: 2019-05-06T02:05:20+08:00
lastmod: 2019-05-06T02:05:20+08:00
draft: false
keywords: [os.Pipe()]
description: ""
tags: [go]
categories: [伪技术相关]
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

> 记一个有趣的小bug

## 0x00 起因

 (紫薯布丁) 因为最近在研究mgo和mongo官方的驱动，因为不能让领导觉得我是在划水，所以，肯定要出点研究成果的。唔姆，那么势必要对mgo和mongo-go-driver和我改过之后的mgo在某些方面做个明显的对比，所以，我想了想打算从他们debug模式的log入手来分析他们的行为，按照我之前写Go的经验，这就是一个读取stdout的简单需求嘛，我很快地写出了一版可以用的代码:

    func capture(f func()) (string, error) {
    	rescueOut := os.Stdout
    	r, w, err := os.Pipe()
    	if err != nil {
    		return "", fmt.Errorf("capture: %s", err.Error())
    	}
    	os.Stdout = w
    	f()
    	out, err := ioutil.ReadAll(r)
    	if err != nil {
    			log.Fatalf("capture: %s", err.Error())
    	}
    	if err := w.Close(); err != nil {
    		return "", fmt.Errorf("capture: %s", err.Error())
    	}
    	os.Stdout = rescueOut
    	return string(out), nil
    }

逻辑看上去也非常简单 ，先暂存之前的stdout，然后再把writer指向stdout。这样就可以从reader里面把stdout全部读出来了，之后再恢复之前的stdout。

这代码看上去确实没啥大问题，也能很漂亮地完成需求，直到。。。。

## 0x01 问题他来了！

(紫薯布丁)  一次机缘巧合之下，我打开了mgo全部的调试信息，想观摩下 mgo 的 节点选举 过程。`go install`,  `mongotest`, 然后看到了一串panic信息。`goroutine deadlock` ？？？从代码字面上看根本看不到有地方开启了goroutine ,  (经过一段激烈而又紧张的小黄鸭调试)，基本确定了问题就出在了这个`capture`,  难道是我的姿势不对？可是我之前都是这么写的啊。

带着这个问题，我开始面向Google debug，看到了基本所有有关的示例和开源项目的代码都是这么写的啊，直到我找到了一个golang/go的一个[issue](https://github.com/golang/go/issues/13142)(不得不说我的运气是真的好)。然后Go Team的大佬回复到:

> gzip.NewReader tries to read the data from the pipe. At the time you call gzip.NewReader, there is nothing writing to the pipe and there are no other goroutines running. That is why you get that crash. The fix is to move the call to gzip.NewReader after you start the goroutine writing to the pipe.

按照这个issue的说法，其实这个问题就也可以迎刃而解了。针对这个问题，我大概理了下原因，在stdout过大的情况下，`ioutil.ReadAll(r)` 这里的r其实是个空的reader，并且一直在等待stdout塞到pipe里面去(也就说这个时刻pipe里面确实什么都没有)，这时候因为也没有其它的goroutine在跑，就导致了这个deadlock的出现。

## 0x02  涨姿势了!

(紫薯布丁)  参考着那个issue的写法， 我开始对我的代码也进行改造。思路是这样的：既然因为“这时候因为也没有其它的goroutine在跑”，那我自己造个阻塞的channel不就好了，这个channel只要撑到stdout全部写到pipe里面去就可以功成身退了，按照着这个说法，我尝试着写了一版fix:

    func capture(f func()) (string, error) {
    	rescueOut := os.Stdout
    	r, w, err := os.Pipe()
    	if err != nil {
    		return "", fmt.Errorf("capture: %s", err.Error())
    	}
    	os.Stdout = w
    	outC := make(chan string)
    	go func() {
    		out, err := ioutil.ReadAll(r)
    		if err != nil {
    			log.Fatalf("capture: %s", err.Error())
    		}
    		outC <- string(out)
    	}()
    	f()
    	if err := w.Close(); err != nil {
    		return "", fmt.Errorf("capture: %s", err.Error())
    	}
    	os.Stdout = rescueOut
    	// 在stdout没有全部写到pipe之前，让outC阻塞住
    	return <-outC, nil
    }

oh ! nice ! 问题终于解决了。

## 0x03 问题总结与抽象

简单地写了几个playground，可以拿来玩玩:

- [stdout很小，所以没有遇到问题](https://play.golang.org/p/M2fRPi8LErR)
- [增加stdout，使得代码可以复现deadlock](https://play.golang.org/p/NtpJkCEXDX)
- [修复deadlock](https://play.golang.org/p/IllaJSzVgWw)

以上，随手记录工作中的乐趣。
