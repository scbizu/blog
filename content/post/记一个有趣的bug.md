---
title: 记一个有趣的bug
categories:
- 伪技术相关
tags:
- go
---

## 先看代码

> [Go Playground](https://play.golang.org/p/FbDGB-h5RP8)

```go
package main

import (
	"fmt"
)

type bug struct {
	name  string
	cause string
}

func main() {

	bugNames := []string{"an", "interesting", "bug"}

	var bugs []*bug

	bugObj := &bug{
		cause: "sth",
	}

	for idx, bn := range bugNames {
		bugs = append(bugs, bugObj)
		bugs[idx].name = bn
	}

	for _, b := range bugs {
		fmt.Printf("bug:%v", *b)
	}

}


```
其实,不难看出,这段代码想要的结果是:`bug:{an sth}bug:{interesting sth}bug:{bug sth}`,但是结果却变成了:`bug:{bug sth}bug:{bug sth}bug:{bug sth}`

## why ?

可能熟悉指针指针的同学一眼就看出来了,这么写是显然不合理的,因为在循环外部的`bugObj`这里不是一个结构体,而是一个结构体指针,所以,在循环里面`bugs[idx]`其实操作的一直是同一个`bugObj`,也就是说,这边的更新操作是不成立的。

具体点说,可以尝试说明下这个循环里面都发生了什么:

* idx == 0; bugs[0].name = "an" => `[]*bug{&bug{name:"an",cause:"sth"}}`  --- √
* idx == 1; bugs[1].name = "interesting" => `[]*bug{&bug{name:"interesting",cause:"sth"},&bug{name:"interesting",cause:"sth"}}` --- x(在这次操作中，`bugs[0](idx == 0)`和 `bugs[1](idx == 1)`其实指向的是同一个`bugObj`)
* idx == 2; bugs[2].name = "bug" => 同上

## 改正

改正其实也很简单,不改变原需求的情况下,把初始化bugObj那段代码移动到循环里面来就是了。

也就是:

> [Go PlayGround](https://play.golang.org/p/RRJZcZInSds)

```go
package main

import (
	"fmt"
)

type bug struct {
	name  string
	cause string
}

func main() {

	bugNames := []string{"an", "interesting", "bug"}

	var bugs []*bug

	for _, bn := range bugNames {

		bugObj := &bug{
			name:  bn,
			cause: "sth",
		}

		bugs = append(bugs, bugObj)

	}

	for _, b := range bugs {
		fmt.Printf("bug:%v", *b)
	}

}

```

## 随想

滥用指针不是什么好事情,有指针机制可能会让语言很灵活,但是用之前(至少在Code Review的时候)也要先想想清楚,之前跟SY大佬(和Go TG Bot API群的同学)讨论过在TG BOT SDK里面很多地方传的都是结构体数组,而不是结构体指针数组的问题,得到的回答是`it is enough`。确实是让人无法反驳但是又没啥错的回答呢。虽然这不关上面的那个bug啥事,但是只是想说 **好用的东西也不要去滥用**。

以上。来自一只很久很久没有更新博客的菜鸡。
