---
categories:
- 伪技术相关
- 算法
- 工作点滴
tags:
- in action

title: 从遍历DOM树谈BFS和DFS
---

### 工作点滴(闲谈)

这是工作点滴目录下的第一篇文章,上班确实累,老大其实也没怎么带我,就说了几个需求,让我自己去完成,最后完成帮我做次Code/Mind Review ,然后下一个任务。但是我觉得这个礼拜我成长了很多。应该是环境的关系吧,边上的人都在认真写代码,你就不会想去怠慢。而(普通本科)学校里根本没有这种驱动力,所以,我觉得`环境`对一个人的塑造作用比本身的`自制力`要强的多(不考虑孤僻症的特殊人群).

### 正题
这几天一直研究`爬虫框架`,由于要做某些页面的分析,去拿`<form></form>`里的一些key(e.g. 就像input的name这种)用做后期处理。写过HTML的应该都知道DOM结构,然而并不是`<form>`这个tag下就有`<input>`给你拿的 每个页面都不一样  有的会在外面套好几层`<div>`.所以要做的事就很明显了 **遍历这块DOM** .

由于[go-spider](https://github.com/hu17889/go_spider)框架内置了`goquery`(一个类似jQuery的golang DOM处理包) ,所以我也就顺便用goquery吧。仔细思考,会发现这个问题其实是`遍历一个多叉树`。

怎么遍历树 ? 我记得大一的课堂上说过 `深度优先遍历`(DFS) 和 `广度优先遍历`(BFS).

* DFS:

深度优先遍历,顾名思义,从Root节点 开始 沿着子节点一直遍历下去。直到子节点遍历完成 回溯到上一个节点 继续此过程。

我的思路是 上个节点先去获取字节点的`Length`  `if Length!=0` 那就`Each`这个这些子节点。然后递归这个过程。最后实现整个DOM的遍历。

```
//GetChildsWithTag  find data traversely
func GetChildsWithTag(tagname string, s *goquery.Selection) []string {
  var tags []string

  s.Find(tagname).Each(func(i int, child *goquery.Selection) {
      //Children numbers
    len := child.Children().Length()
    tag, _ := child.Attr("name")
    tags = append(tags, tag)
    if len != 0 {
      GetChildsWithTag(tagname, child.Children())
    }
  })

  return tags
}
```

* BFS

那么 广度优先遍历 也就很好理解了,先遍历ROOT节点,然后**一层一层**遍历下去....

我的思路是 `Each`了`ROOT`之后 子节点如果也有Children的话 把子节点的Children放在一个`Selector`里面。第一次`Each`之后，把这个`Selector`作为`ROOT`,递归这个过程。

我没有对这个方法进行实现  有兴趣的 可以试一下 ~  :)


显然 比起来 DFS的方法比较容易想到  也比较容易实现。 但是碰到单层节点很多的情况 还是选择__BFS__比较好。__DFS__的思想在爬虫中应用面还是很广的。
