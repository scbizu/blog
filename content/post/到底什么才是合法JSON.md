---
categories:
- 伪技术相关
- golang
tags:
- in action
- golang
- json
title: 关于JSON的思索
---

## JavaScript Object Notation
### 简述
JSON是这几年一下子火爆起来的数据交换格式,在前后端的交互中,`JSON`也似乎变成了一门必修课,相比之下,反而像`XML`之类的格式在数据交互中已经很少听到了。当然,`JSON`会火起来,大抵是因为 __轻__ 和 __易读__ 吧。前端的同学和后台的同学都能很快的上手(大概5分钟就能掌握其基本的语法格式)。

### 以前的认识

由于大二做项目的时候，IOS的学长说你给我的接口的数据不要用数组啊 要用`JSON`的格式给我 这样我们两都方便。那时候 还第一次听说`JSON`。 之后 一直在赶项目进度, 所以 ,那时候理解中的`json` 无非就是 `$jsondata=json_encode(array)` 只要把数组的结构封装好 扔给前端 就万事大吉了。以至于 一直到前些日子 还是这么认为的，唯一的区别大概就是换了种写法(在golang中通常都是`Marshal`一个`Struct`)。从未去了解过 到底怎样的才算`JSON` 一定要是有`“{}”`的？抑或是一定要带着`”：“`的？

### 说说今天的事

应该是昨天的事了，实习带我的老大看我爬虫撸得差不多了。就凑过来说要给我出道题,让我打开了一个`Online JSON Formatter`,说了句”__随便输串JSON试试__“  当然 我习惯得输了`{"a":"a","b":["a","b"]}`。 然后 Run 了一遍。老大说了句 把它用golang的terminal实现出来吧。。 我在想这还不简单？ 于是昨晚回家 我就很快的用了`cli`包 和 `json`包 实现了这个功能  后续 还 ”锦上添花“地加了read unformatted json from file(其实是个意外 偶然发现cli的包竟然自带了read json from file，叫[JSONIndent()](https://godoc.org/github.com/mkideal/cli#Context.JSONIndent),于是很巧得就用上了)和download ouput to file。以为早上去上班的时候 应该老大会表扬我的吧。。。毕竟一个小demo就做得这么精致，然而。。。。。 又是一顿语重心长的教导:"别把这个搞得这么华丽，你告诉我你格式化JSON的逻辑是怎样的?" "啥?逻辑?不就是一个自带的`json.Indent()`吗" ”我的意思是让你自己去实现这个逻辑。。“ 于是 只能打回去自己写`Indent()` ,这时 因为要自己写这个`Indent()` 所以 搞清楚到底JSON是怎么一回事 就变得 非常重要了。 当然 正常coder的第一反应 肯定是去查 `Indent()`的源码  关于真正format的逻辑 其实很简单 就跟 学校每门语言入门时在`循环`这一章节里提到的 星号正/倒三角 类似的题 花了大概十几分钟就把`Format`的逻辑写完了 真正感觉自己难以接受的是源码里的那个 `scanner`的实现。很难理解为什么首字符为什么要判断这么多字符。于是 去了[json.org](http://www.json.org/json-zh.html) 终于解开了心里的疑虑。都是 定性思维 的锅啊！ JSON明明还有`数组(An ordered list of values)` ,`字符串(string)`,`数字(number)`的啊。。。

### scanner.go中判断首字符的函数

```
// stateBeginValue is the state at the beginning of the input.
func stateBeginValue(s *scanner, c byte) int {
	if c <= ' ' && isSpace(c) {
		return scanSkipSpace
	}
	switch c {
    //判断的是 k-v 的情况
	case '{':
		s.step = stateBeginStringOrEmpty
		s.pushParseState(parseObjectKey)
		return scanBeginObject
    //判断的是 数组 的情况
	case '[':
		s.step = stateBeginValueOrEmpty
		s.pushParseState(parseArrayValue)
		return scanBeginArray
    //判断的是 字符串 的情况
	case '"':
		s.step = stateInString
		return scanBeginLiteral
    //判断的是 数字(负数) 的情况
	case '-':
		s.step = stateNeg
		return scanBeginLiteral
    //判断的是 数字(小数) 的情况
	case '0':
		s.step = state0
		return scanBeginLiteral
    //判断的是 true(boolean) 的情况
	case 't':
		s.step = stateT
		return scanBeginLiteral
    //判断的是 false(boolean) 的情况    
	case 'f':
		s.step = stateF
		return scanBeginLiteral
    //判断的是 空json 的情况    
	case 'n':
		s.step = stateN
		return scanBeginLiteral
	}
    //判断的是 数字(正数) 的情况  
	if '1' <= c && c <= '9' {
		s.step = state1
		return scanBeginLiteral
	}
  //json所有格式就这些 没有的话 就只能return error了。
	return s.error(c, "looking for beginning of value")
}
```
> UPDATE :

自己也撸了个 [json.Indent()](https://golang.org/pkg/encoding/json/#Indent)的 [demo](https://gist.github.com/scbizu/3a9b12a822a2563bc406f9aeaae48969)  有兴趣的可以看看哈～


## end
