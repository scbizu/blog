---
categories:
- 伪情感相关

tags:
- 总结

title: 下一站,何去何从
thumbnail: https://ws1.sinaimg.cn/large/005uxZXHgy1fd7ve5a4svj30co09e0t1
---
> 终于决定还是离职了,大概是对自己无能的一种默认吧。

### Bye，NDASEC

其实早在某短离职的时候 我也想过是不是我也该换个坑。但是,却迟迟没有下决心。与其说公司有go的相关技术栈不如说只是领导们的心血来潮,其实在杭州想组一支写Go的团队挺难的吧。11月的时候跟公司一起写Go的同事谈过我是否该走的话题,同事说再留一阵子呗,现在这个同事自己都走了还是有点讽刺的,大概也是觉得这样下去没什么提高吧。所以 过年上来 我也下定了走的决心 但是 领导说未来想把一些以前Java的代码用Go重构,意思大概是虽然Gopher都走光了,但是未来还是可以抢救一下的意思吧,领导也一直在公司内部带Golang的节奏 鼓励内部从C++/PY猿们转到Go上来,但是几乎没有成效。

之后,就打定了走的决心,2月中旬开始各种投简历,但是拒的拒,还有很多也是石沉大海。2月真是郁闷的一个月啊。正当我打算索性辞职回校闭关的时候,Yeeuu,ezbuy,CaiCloud相继发来面试邀请。

虽然 不一定拿得到offer 但是还是从心底里感谢这几家的赏识。

最后,虽然迫于无奈离开了NDASEC,但也非常感谢老东家的信任和器重。

### Hello, New World

> 今晚,风很大。所有的面试大概就告一段落了吧,那就整理下找工作至今的面试吧

* NDASEC

  这大概算是正式的第一次面试了,总觉得这份实习的面试可能是个假面试,那次面试的时候老大还在,看着json包都不会用的我,最后会要我这个实习 我一直觉得可能走了狗屎运吧。(那时候正式转Go才不到一周) 就被随便问了OOP的问题 和 Go的channel机制。

* Yeeuu

  对于这次面试,体验还是挺不错的吧。由于我把简历挂自己的站上了,HR还特地把我的简历打印出来233 这次面试内容基本就是按照简历来走的 问了很多我在简历上写的知识点,包括 HTTP协议 TCP/IP 讨论了golang的`interface{}`的用法 也问了Golang里Goroutine和channel相关的知识点。最遗憾的是 我算法题竟然没答上来 !算法题是求 **数组的最大差值**,现在想来其实很简单,只需要一次排序就可以解决了(`MaxDelta=Max(arr)-Min(arr)`),这样完全可以在O(logn)完成的。还好 还是拿到了实习和转正的Offer 不然真的会很遗憾 。

* CaiCloud

  CaiCloud是从算法面开始的,直接给的一道算法题 。题目是 **求一个字符串的最长不重复子串 只要求给出子串的长度**  想了很久都想不出O(n)的解法 。只想到了一种递归解法 通过不断缩小这个字符串的长度 然后递归出解 代码如下:

```go
package main

import "fmt"

func loopString(strs []byte) int {
	maxlen := 1
	curflag := 0
	for index, curbyte := range strs {

		if len(strs) == 1 {
			return 1
		}

		if index > 0 {
			for i := curflag; i < index; i++ {
				if curbyte == strs[i] {

					if maxlen >= loopString(strs[index:]) {
						return maxlen
					}

					return loopString(strs[index:])

				}
			}
		maxlen++
		}
	}
	return maxlen
}

func main() {
	testString := "abbcad"
	testBytes := []byte(testString)
	fmt.Println(loopString(testBytes))
}
```

然后 就没有然后了 估计是跪了。

* ezbuy

  这次面试大概是面试以来觉得自己最菜的一次吧，如果能用一句话来概括这次面试,那就是“我好气啊！” 明明以前自己认为一辈子都不会忘记的知识点就是死活回忆不起来,这就证明了写笔记 整理知识点有多重要了！印象最深的大概是两个知识点 **TCP的滑动窗口** 和 **实现hashmap**  滑动窗口被提示了是用在等待ACK的时候的还是想不起来 看来是要补补[什么是滑动窗口](https://my.oschina.net/xinxingegeya/blog/485650)了  .实现Hashmap想到了数组 想到了动态数组 竟然又没想到链表数组。还有CTO面里面,由于真的好用没用泛型了 泛型的问题都答得很水。感觉这个面试也已烷。

## 叉路

> 趁着年轻 出去闯闯

最后,得到了`Yeeuu` 和 `ezbuy`两家的offer ;

因为喜欢挑战和刺激 我大抵会选择在上海的`ezbuy`的吧;

但是 `Yeeuu`也确实是个很有潜力的团队 :) 如果 当初找实习的时候 多看看就好了。

我也不造这条路 通向哪里 ;

与我而言 如果能让自己变得更强，那么就是值得的;


以上。。。
