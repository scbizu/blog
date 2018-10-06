---
categories:
- 伪技术相关
tags:
- golang
- proxy
title: 天朝局域网内go get的正确姿势
---

## 前言

Golang获取包最简单的方法就是直接执行`go get xxxx`,与此同时,这也是Golang被诟病很多的一点 非常不利于版本管理 个人的工作环境可能会因为引包不对而被污染  所以后来就有了`govendor`  `gopm` 等关于go的包管理器 。当然 这不是今天要说的内容。今天要说的是关于墙的解决方案 .很多Golang开发者大多会遇到这个问题:**go get 包的时候会卡死 或是直接超时**  因为我们伟大的防火墙的存在 很多包都是不能直接获取的。。然后 有人会用`Shdowsocks ` + `proxychains/proxychains-ng` 作为Terminal xx 的工具 配置好之后 经过测试发现proxychains能用  但是go get 还是卡死了。这个事情已经有中国开发者在Github上对Go Team提问过了,Go Team的反馈是不能单单为了中国改变go get的实现模式,详情issue请戳[这里](https://github.com/golang/go/issues/18508) 。

## polipo

> 关于[polipo](https://wiki.archlinux.org/index.php/Polipo_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)

Polipo 可以把`Socks Proxy -> HTTP Proxy`

### 0x00: 安装&&配置

  * Ubuntu及其衍生版下可以使用 `sudo apt-get install polipo`
  * 在`/etc/polipo`下找到config的配置文件 ,配置如下:

```shell
    # This file only needs to list configuration variables that deviate
    # from the default values.  See /usr/share/doc/polipo/examples/config.sample
    # and "polipo -v" for variables you can tweak and further information.

    logSyslog = true
    logFile = /var/log/polipo/polipo.log

    proxyAddress = "0.0.0.0"

    socksParentProxy = "127.0.0.1:1080"
    socksProxyType = socks5
    proxyAddress = "::0"        # both IPv4 and IPv6


    chunkHighMark = 50331648
    objectHighMark = 16384

    serverMaxSlots = 64
    serverSlots = 16
    serverSlots1 = 32

    proxyPort = 8123
```

  * 然后 执行 `sudo polipo -c /path/to/polipo/config` (/path/to 为polipo父目录)

  * 开启 Http Proxy ` sudo polipo socksParentProxy=localhost:1080`
  >  此步建议在第二步做 :)。

### 0x01 测试 使用

* 加上http(s)_ proxy来获取包 `http(s)_proxy=http://127.0.0.1:8123 go get -v -u github.com/xxx/xxx`
* OR
```shell
$ export http_proxy=http://127.0.0.1:8123
$ export https_proxy=$http_proxy
$ go get -u -v github.com/xxx/xxx/
```
## 不怎么优雅的姿势

* 从国内的镜像站(例如 gopm.io)下下来之后 再通过`go install`来安装包  但是这样会丢失git记录 不方便包的更新。

* 直接git clone 到本地.

* 通过`Shadowsocks` + `proxychains4` + `govendor` 的方式来获取包因为用到的东西 比较麻烦 所以也不是怎么优雅吧 但是强烈建议用类似`govendor`的包管理工具来进行golang package的管理！


## Reference

* [GitHub](https://github.com/golang/go/issues/18508)
* [ArchLinux Polipo](https://wiki.archlinux.org/index.php/Polipo_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)  
* [Golang中国](http://golangtc.com/t/582d3411b09ecc08ce0003c2)
* [codevoila](http://www.codevoila.com/post/16/convert-socks-proxy-to-http-proxy-using-polipo)
