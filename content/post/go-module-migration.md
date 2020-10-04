---
title: "Go Module Migration"
date: 2020-10-04T19:00:06+08:00
draft: false

keywords: ["Go Module"]
description: ""
tags: ["Go"]
categories: ["工作点滴","Go"]
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

# Go Module Migration From GOPATH Vendor

> 本篇记录了我们团队是怎么从「特殊」的Go Vendor模式迁移到Go Modules，并且怎么在每个项目中管理这些依赖

## Vendoring

在介绍Go的vendoring机制之前，我们可以顺便考下古，后退到Go还没有包管理概念的蛮荒时代。经历过那个时代的Gopher只有自己知道怎么编译自己写的代码，没有版本管理也不保证兼容，如果你想编译，那就要把依赖一个一个go get到本地。当然这样开发起来费时费力，实在配不上Go简洁高效，大道至简的作风，于是Go Team接受了社区的建议，引入了[Vendor](https://go.googlesource.com/proposal/+/master/design/25719-go15vendor.md)的概念，允许代码仓库把自己的依赖通过vendor directory放在当前仓库内，但是在官方的说明中我们可以看到一直是标注为「Experiment」的，并且也高亮了主要的机制:

> If there is a source directory d/vendor, then, when compiling a source file within the subtree rooted at d, import “p” is interpreted as import “d/vendor/p” if that path names a directory containing at least one file with a name ending in “.go”.
When there are multiple possible resolutions, the most specific (longest) path wins.
The short form must always be used: no import path can contain “/vendor/” explicitly.
Import comments are ignored in vendored packages.

根据当年的[proposal](https://docs.google.com/document/d/1Bz5-UB7g2uPBdOx-rw5t9MxJwkfpx90cqG9AFL0JAYo/edit)，之所以引入vendor mode ，并且命名为「vendor」，有以下几个原因(摘自原Google Doc):

- 当时社区使用比较广泛的包管理工具[gb](https://github.com/constabulary/gb)使用了vendor作为依赖管理的文件夹名。
- 「external」这个名字看上去虽然与 Go 的 [internal 机制](https://golang.org/s/go14internal)相对，但是用在这里意义有点不明确。
- 社区的大家都喜欢用vendoring来描述依赖管理 ( ？)

并且根据当时的设计，Go tool 会对 vendor目录会进行层级搜索：也就是 当前目录 → **GOPATH** → GOROOT  (参考 [golang/go/build 的  searchVendor 方法](https://github.com/golang/go/blob/master/src/go/build/build.go#L620-L646))

当时的社区大多数的项目采用第一种方式，也就是在当前目录挂上vendor目录，这对像GitHub这种开源社区来说是足够的了，大多数项目都不是org级别的，只要管理自己项目的依赖就可以了。但是我们一旦我们把这种设计引用到公司项目群级别可能就会比较头疼了，每个项目都有自己的vendor依赖，这些依赖一旦设计到一些基础组件的sdk package，这对维护基础组件的同学来说简直就是噩梦，我们根本不知道那个业务线用的是哪个out of date 的sdk 。为了统一第三方包，我们引入了GOPATH级别的Vendor，自己用Shell写了些小工具通过 git subtree 的形式维护了一个vendor的仓库，跟vendor同GOPATH的项目的依赖都会收拢到这个vendor仓库。这个改造在当时来说是非常值得的，也成功帮助我们在 全部服务升级grpc版本 ，升级Consul 等技术改造方面避了很多坑。（众所周知，grpc-go版本升级的Breaking Changes是可以让人大喊 真～不～戳 的

但是自从Go 1.10 时候 rsc 的 [Go & Versioning(vgo)七连](https://research.swtch.com/vgo)的出现，以及rsc跟sdboy在Twitter上长达数十个Thread的激烈讨论（sdboy是Vendoring的支持派，并著有golang/dep，一度引领Go社区的版本管理风潮 ），我就知道风向变了，我们得想个办法把现在的体系迁移到Go Module 。

## Moduling

翻遍 [Migration Guide](https://blog.golang.org/migrating-to-go-modules) ，最后只在角落里面找到了，Go tool 的`go mod vendor`不支持除了当前目录vendor之外的模式（翻了源码，发现也确实不支持）。有点不理解为什么既然vendor支持三种路径的vendoring，但是`go mod vendor`竟然只支持一种，也许是Go Team加入`go mod vendor`本来就是像社区的一种妥协吧，或者是像群友说的Google没有这种需求hhh。

在Gopher Slack上问了圈，没人理我之后，我就打算自己搞这种场景（GOPATH Vendor）的支持了。（顺便还能体验下对 Go Module本身的开发，等我写完了就去golang/go 开issue

### ezmod #1

第一个版本的实现我在团队内部的wiki上写了design guide，得到了团队内部的支持之后就开始开工了。

这份design guide的结构抽象一下会分成三个部分:

- 从我们的vendor导出来的index文件，这个文件跟以前vendor盛行时代的 `vendor.json` 或者 `vendor.toml` , `vendor.yaml` 这种文件很像，描述的是我们当前依赖的 **引入路径(import path)** 和 **依赖版本号(version)，**跟 `go.mod` 里面描述依赖的形式也一致。
- 第二个部分其实是很脏的部分，他描述了开源社区的一些breaking changes，比较著名的可能是 logurs的作者改名 事件，导致different import path，需要自己手写replace矫正。还有我们以前的陋习 —- 魔改vendor ，这是非常差的习惯，会对技术栈的可维护性造成毁灭性的打击，非常不推荐(fork的形式会优雅很多)，这种我们也需要手写replace规则把这些依赖搬出来单独维护。
- 第三部分也是最核心的部分，我们需要写一个工具，根据 **第一部分的version index文件** 和 **第二部分的replace规则** 组成新的 `go.mod` 。

依靠着 Go Team 的 [x/mod](https://github.com/golang/mod) ，编码并不是很难。我们最后得出的 `go.mod` 结构大概是这样:

![mod_layout.png](https://i.loli.net/2020/10/04/goK9kJEflbPneDh.png)

图中的 `ezmod.yaml` 就是Part 2 定义的 replace rule，我对于Part 3中的mapping的具体做法是，先进行go get生成完整的 `go.mod`，有 Part 2 的帮助，应该可以成功生成 `go.mod`。再从index中根据import path找出对应的version，这边有个trick ，我们从vendor index拿到的 SHA-1 是不能直接用来replace的，需要通过 `go get import_path@version` 的方式拿到完整的version 。（这是我从[k8s pin  module的脚本](https://github.com/kubernetes/kubernetes/blob/master/hack/pin-dependency.sh)中借鉴(抄)来的 ，后来意识到好像可以用 `x/mod/semver` 直接糊，还少了很多次 Network I/O ）

OK，花一天写完了这个Generator (大部分事件在debug)，拿我们基础组件仓库做了个实验，确实replace 规则都按照预期生成了，看起来也很完美，commit module, 打好tag，cc 组内同事，下班搓炉石！

### ezmod #2

第二天组内小伙伴用这工具给业务项目打mod的时候，遇到了一个问题:

> 业务项目依赖基础库的时候，好多三方库的version都不对，因为API Changes的关系，编译都无法成功。

查了问题之后发现，基础库的module replace规则都没有生效。我们做了一个最小的重现case，果然，依赖的 `go.mod` 里面的replace 规则会被忽略。也就是说:

```bash
A -> B -> C1 // A依赖B，B依赖C,但是B的go.mod把C替换成了C1
A -> B -> C // A 中不会使用B中go.mod中的replace rule，B依旧依赖C
A ->replace C => C1 -> B -> C1 // 只有A中的replace rule中也增加 C=>C1，A中才会使用C1
```

这个问题解决方案其实也不难得出：我们的mapping 要改，在全部项目里面根据vendor index全量生成replace规则就可以了。但是对于我们几百个子仓库的vendor来说，全量生成非常耗时。于是，我们开始思考新的解决方案。

「如果我们已经知道了依赖的版本，我们直接require不就好了，为什么一定要replace」，同事的这句话倒是给我提供了新的思路。于是，开始改写部分代码把vendor index里的依赖全部require到了module里面。但是原 本的replace版本真的一无是处了吗 ？我倒是觉得未必。

在go mod的开发中，replace 经常被用来作为本地 module 的替换，可以用来作为debug或者用来做本地测试。结合这点，我们如果可以把之前的模式跟这个场景结合起来，提供一个本地develop模式。对应的具体场景可能是： 1) 我们可以方便得找到项目中的依赖，并把它replace成某个还没有正式发布release tag的依赖。2) 只要我们require也是根据vendor index生成的，那么这个replace只在当前仓库中生效，这样即使在联调环境我们也还是可以使用develop模式下的module，来应付一些基础库在联调环境下的差异化改造。 3 ) 真正发布(生产/测试)时，需要CI移除掉之前的module，并生成一份新的publish module，这个module里面会把vendor index replace规则删掉，使用 stable 的依赖。

通过这种方式，我们把开发分成了两种角色：

- 依赖提供方(publish):  作为依赖提供方，module描述的依赖一定是唯一的（当然这里不包括之前讨论过的全局的replace规则），也就是说，依赖一定是要require的，不能通过replace把一个require依赖fix成另外一个版本。
- 依赖使用方(develop):  作为依赖使用方，我们是可以把口子放开的，因为开发在日常开发时，可能会选择某些依赖的full log 或者 debug 模式。

这样就还是可以保证我们的所有生产/测试环境(publish)的依赖还是可以在vendor里面收拢的，不会又变得不可控。

## 总结

其实看得出来，我们这次改造是为了迁移mod而迁移到Go Module，真正的依赖管理还是会依靠vendor和vendor index，我们更新某个三方依赖时，还是需要去把它merge到vendor里面，再通过vendor index提供给外部项目的Go Module中使用。相比之前还是有些优势的，之前有新人入职配置环境或者我们配置CI pipeline时，由于vendor这个东西实在有点大，流程会卡在下载vendor上面非常久的时间，现在只要通过 `ezmod` 去拉取一份vendor index文件就好了，顿时从几十GiB下降到了几十KiB。同时也能继续保证所有依赖能在一个地方(vendor)统一管控。

这样方案也可以看作是对 `go mod vendor` 的一个扩展补充吧。

以上。
