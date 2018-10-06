---
categories:
- 伪技术相关
- golang
tags:
- crawler
- 脑洞
- HTTP/1.1
title: 有可能是一种思路不一样的网站内容更新的监测方案
---

## Etag(实体标签)

> ETag是一个不透明的标识符，由Web服务器根据URL上的资源的特定版本而指定。如果那个URL上的资源内容改变，一个新的不一样的ETag就会被分配。用这种方法使用ETag即类似于指纹，并且他们能够被快速地被比较，以确定两个版本的资源是否相同。ETag的比较只对同一个URL有意义——不同URL上的资源的ETag值可能相同也可能不同，从他们的ETag的比较中无从推断 。                   -- From  wikipedia

在网页内容检测方面, 可能这是个思路不一样的点 。虽然大多数的网站采用的是 `cache-control:no-cache` `Date/Expire` 的缓存过期方式 ,但是如果在有`etag`的场合 通过`etag`的方式去实现获取可以得到比较好的效果. 这样就可以不去分析`body`体 而在报文分析阶段直接处理。

最大的缺点 大概就是 相比于其他的检测方案来说 这个方案 显得**太不稳定了,可以认定是一种投机的做法了**


## Steps

* 第一次`Get`获取到`Response`  拿到这次`Response`的`etag`
* 第二次`Get`到目标网址 带上 `If-None-Match:%第一次的etag%`
* 判断: 如果是`200` 则是新的资源了  如果是`304` 那就说明跟上一次的一样  没有改变过 。

## Example

> Take golang as an example

```go
package main

import "net/http"

func main() {
	res, err := http.Get("http://www.infoq.com/cn/articles/etags")
	if err != nil {
		panic(err)
	}
	etag := res.Header.Get("etag")
	println("Etag:", etag)
	c := new(http.Client)
	req, err := http.NewRequest("GET", "http://www.infoq.com/cn/articles/etags", nil)
	if err != nil {
		panic(err)
	}
	req.Header.Set("If-None-Match", etag)
	resp, err := c.Do(req)
	if err != nil {
		panic(err)
	}
	etag2 := resp.Header.Get("etag")
	println("Etag2:", etag2)
	println(resp.Status)
}
```

```shell
Etag: W/"96734-1484829603000"
Etag2: W/"96734-1484829603000"
304 Not Modified
```

> 但是如果把infoq的改成v2ex的网址  就会发现一直是200 OK ... 即使资源没有Expired掉


以上
