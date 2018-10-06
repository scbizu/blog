---
categories:
- 伪技术相关
- golang
tags:
- in action
- golang
title: golang结构体和json之间的转换小结
---

golang提供了结构体到特定格式json/xml/... 转换的便捷途径

我们通过json包 很方便的就可以进行struct到json转换。

举个栗子:

现在有这么一个结构体:



    type JsonFor struct {
        Foo   string `json:"foo"`
        Foo_s string `json:"foo_s"`
        Foo_t string `json:"foo_t,omitempty"`
    }



结构体字段的**字段(field)**最后的一串(string 之后的)叫做**标记(tag)**.

双引号中的字符串是转换成json之后的 json字段名(key)

(当然 也可以不加这个tags 那么json之后的会是default key 也就是你的字段名(filed name)

下面说一下 这些tags的用法(optional comma and options):





  * `什么东西都不加`：上面说过   不加tag  struct 的filed name就是**json的default key**。


  * `"-"`: e.g:`json:"-"` 这个tag会导致该field 会被当前的package忽略掉。


  * "(string)":e.g.:`json:`json:"foo"` 这个tag会导致转成json后 key变为**foo  ,而不再是Field name(default key)**`


  * "(string),omitempty":在上一个tag上面添加了一条规则->如果当前字段为**空值(empty)  那么忽略这个字段**


  * ",omitempty":在第一条规则下添加->如果当前字段为**空值(empty)  那么忽略这个字段**


  * ",string":只用于以下类型:string, floating point, integer,  boolean  。字段值(value)也会被json编码。(常用于与JS的交互)





由于书上都没有介绍  这边记录下。。。。

实验代码在gist上。。。。
