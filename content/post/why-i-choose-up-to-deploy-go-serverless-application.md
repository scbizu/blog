---
title: "使用up来部署AWS Lambda应用"
date: 2019-10-05T23:50:21+08:00
lastmod: 2019-10-05T23:50:21+08:00
draft: false
keywords: ["Go","Serverless","FASS"]
description: ""
tags: ["Go"]
categories: ["伪技术相关"]
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

## 前言

> up真香

终于还是来写这篇博客了，本文记录了我的AWS Lambda Function APP 的部署工具从[apex up](https://github.com/apex/up)迁移到原生的`AWS Lambda Go SDK`，然后再迁回``pex up`的整个过程以及原因，希望给之后喜欢折腾AWS Lambda的同学一点经验和灵感。

本博文不会涉及为什么选择Lambda这种Serverless的方案来部署应用的话题，有兴趣的同学可以自己去调研;)


## 什么是apex up ?

up是TJ大佬写的一个AWS Lambda 部署工具，TJ从[apex](https://github.com/apex/up)中抽离了一个workflow, 然后再进行重新封装整合，然后就有了这个完美适配 AWS Lambda 的deploy workflow . up可以通过`up Proxy`把`node`的runtime转发到你的二进制程序(以Go为例)，然后暴露`node`的入口函数`handler`给AWS Lambda 。为什么会是`node` ? 因为up刚开始写的时候Lambda只支持node的runtime。那么为什么之后不换掉`node` 的runtime ? TJ在[这个issue](https://github.com/apex/up/issues/543)里面回答了这个问题。主要是`Go native runtime`和用`node`的`up proxy`在性能上其实也查不了多少，[这里](https://gist.github.com/tshak/7fa2c7803ae5a27f66441752d1bd8e45)有一份benchmark可以看看。当然，随着时间的推移和Lambda runtime的优化，这两个runtime终究会有一个领先，那么到那时候，就可以提PR给up重构代码了。

## up VS Go SDK ?

在接触到up之前，可能你也跟我一样，用着AWS官方的[lambda sdk](https://github.com/aws/aws-lambda-go/)，然后顺利地写出了一个「Hello World」, 但是，当你想部署自己的Web Service或者想迁移自己的Web Service到AWS Lambda的时候，被AWS复杂的API Gateway配置劝退。当然此时的你可能不知道，这之后还要配置监控用的`CloudWatch`和鉴权系统`IAM`。当然，你也可以自己写`CloudFormation`来管理自己的deploy profile，如果你看着几百行的JSON配置不眼花的话。

显然，我们的部署不需要这么复杂，被现代部署工具宠坏了的我们，可能只想写个`Dockerfile`/`YAML`或者一个其它的什么配置文件来帮助我们解决部署的问题，而不是自己去AWS控制台XJB配，（写到这里，可能会联想到[Terraform](http://github.com/hashicorp/terraform) ，现代程序猿可真的是越来越懒了呢)，最后，所有的箭头都指向了 up , up 可以通过一些非常简单易懂的配置来代理你的原生Go程序，这里说的原生Go程序，其实是为了区别用SDK的侵入式写法，如果我们使用SDK来构建并且运行我们的Lambda应用程序，那么代码(`main.go`)中必须显式声明`import "github.com/aws/aws-lambda-go/lambda"`,并且需要调用`lambda.Start()`方法，才能让lambda接入到我们的Go程序中; 但是，代理就不一样，我们不需要去依赖任何关于Lambda SDK的东西，只需要关注自己的业务逻辑就可以了， up proxy 会帮我们与Lambda SDK和API Gateway交互，只需要我们在`up.json`中配置我们需要的代理程序即可，比如:
```json
  "proxy":{
    "command":"./main "
  },
```
> 声明了一个`up proxy`, node runtime的入口函数handler的会代理到这个`main`二进制文件上去

并且，如果你的Lambda Function APP需要暴露一些Web API(也就是需要配置Gateway)，那么必须自己封装一层Proxy来代理你的请求到AWS Gateway里面， 在 Native Go SDK 里面我们可能需要用到类似[gateway](https://github.com/apex/gateway)/[api-proxy](https://github.com/awslabs/aws-lambda-go-api-proxy)这些东西来暴露我们的API，但是在up里面我们不需要关心这个问题，up会帮我们自动生成AWS Gateway的配置代理到你的应用。

综上，up的部署方式**更适合我们Service到Serverless的迁移**,也**更方便Serverless服务的扩展**。

当然，由于多了一个node的壳子，所以Lambda应用的大小会多～3MB左右，但是，AWS的S3的免费空间有点大(Lambda建议不直接上传zip,而是先上传到S3,然后用S3的文件路径挂载到Lambda上面)，3MB左右的大小应该还是可以接受的。


## up在持续集成中的实践
由于up实用的CLI,使得它在集成CI/CD workflow中非常容易。 比如我们的Lambda Function APP有两个环境，一个staging，一个prodcution(apex的`CloudFormation` 也是默认设置了这两个环境)，在程序CI完毕在CD阶段的时候，我们只要调用up CLI的 `up deploy (staging/prodcution)` 就可以触发整个`CloudFormation`的重新“构建”，然后重新生成Lambda的配置，最后重新部署Lambda  APP。

在类似[Drone.io]()这种CI/CD工具中，我们只要定义如下配置，apex就可以在push master的时候，触发production环境的重新部署:
```yaml
- name: up_prod
  image: golang:1.13
  environment:
    AWS_SECRET_ACCESS_KEY:
      from_secret: AWS_SECRET_ACCESS_KEY
    AWS_ACCESS_KEY_ID:
      from_secret: AWS_ACCESS_KEY_ID
    # SERVERLESS: on
  commands:
  - curl -sf https://up.apex.sh/install | sh
  - up deploy production -v
  when:
    event:
    - push
    branch:
    - master
```
如果你还是觉得要写这么东西有点麻烦，那么也许[吴老师的这个Drone插件](https://github.com/appleboy/drone-apex-up)会帮到你。

## up这么方便，为什么我还会有离开up的想法

>  TL;DR : 思维定势，理解上出了偏差

我在开发[feeds](https://github.com/CNSC2Events/feeds)这个项目的时候，偶然发现up的部署方式貌似不支持`GoRoutine`，具体的场景是我有个`GoRoutine`会去更新我的某个cache，但是这会是个比较耗时的Network I/O，所以被我设计成不会阻碍main routine的形式了，但是，这个程序在部署在Lambda上的时候发现这个cache永远不会被更新，表现就像是`GoRoutine`没有被调度一样，但是，在Server环境下确实表现正常，cache很快就被更新了。

于是, 我单独对apex和aws官方的Lambda Go SDK做了[测试](https://github.com/scbizu/gor-test) ，发现无论是up还是aws lambda sdk，GoRoutine和Channel的机制确实都是支持的，难道是up的nodeJS runtime有什么莫名其妙的bug ？

于是，我一边吐槽一边着手基于Go Runtime定制新的workflow(甚至没有给apex提issue，就在Slack冒了个泡，但是没人鸟我)。但是，就在我[写到一半](https://github.com/CNSC2Events/feeds/tree/feature/aws-lambda)的时候，我想到了一个问题， **lambda里面function的生命周期究竟是怎样的 ？**，我们暴露出去的应该是一个function,这个fucntion会被多次并发执行，跟Server的形式不一样，它并没有一个一直在运行的进程来维护资源，所以我们在每次调用某个function的时候，如果有一些需要初始化的资源，那么每次在每个生命周期内都需要初始化，所以，拿我的cache这种全局内存资源举例，每次都要经历初始化的过程，每次的请求更新缓存的机制自然也就消失了，在这种情况下，内存型的cache就失去了他的用武之地。也就是说，在Serverless的世界里，任何可以被更改的全局资源就是个深坑。

正巧又看到了[TJ对于Lambda和内存型cache的说明](https://medium.com/@tjholowaychuk/aws-lambda-lifecycle-and-in-memory-caching-c9cd0844e072)，脸有点疼，这篇文章可以给Lambad的life cycle扫个盲，并且探讨了在Lambda中维护cache在一定程度上的可行性，非常值得对Lambda生命周期一知半解的同学阅读(我彷佛就是在说我自己hhh)。但是由于确实一次函数调用之后，对应的container会封锁background job，我在GoRuotine中定时更新cache的设计确实是不可行的。于是，我在保持cache的基础上，移除了用于更新cache的backend GoRoutine，改成了用`CloudWatch Event`来定时触发我的Function，从而达到了和background GoRoutine差不多的效果。

> 具体可以戳[这个commit](https://github.com/CNSC2Events/feeds/commit/f0b9844b57b6eab565d1f694703934dae8f84711)

那么你基于Go shim的搬运是不是就停止了 ？ **怎么可能，有空了一定继续写！**

以上，记录我对AWS Lambda的折腾历程。Anyway, 我还是挺看好FaaS Platform可以真正被用起来的。

Reference:

* [up Doc](https://up.docs.apex.sh)
* [up with Lambda](https://blog.wu-boy.com/2018/10/deploy-app-to-aws-lambda-using-up-tool-in-ten-minutes/)
* [aws lambda](https://docs.aws.amazon.com/zh_cn/lambda/latest/dg/getting-started.html)
* [Shopify运用Lambda Cache的简单实践](https://rewind.io/blog/simple-caching-in-aws-lambda-functions/)
