---
categories:
- 伪技术相关
tags:
- docker
- elasticsearch
title: Docker下搭建Elasticsearch及其集群
---

## Why Docker ?

作为当下大热的`Docker`,一直有用到工作中的冲动。这次正好有个集群搭建的需求,就毫不犹豫,一脚踏入`Docker`,容器化的概念方便部署,测试。另外一个原因是 需要了解下`ElasticSearch`在集群部署下的操作,没有足够的物理机可以用,于是就用Docker在本机搭建了一个`基于Docker的ElasticSearch集群` 。

## ElasticSearch

[ElasticSearch](https://www.elastic.co/products/elasticsearch)作为玩大数据必备的数据库,集成了一套强大的文档搜索引擎,和类似`NoSQL`的数据库引擎.也许相比 **传统关系型数据库** 和 像`Redis,Mongo`这样的 **NoSQL数据库** 牺牲了存储时的性能,但是`ElasticSearch`强大的文档搜索能力,和天生支持分布式的设计受到了各大爬虫界的青睐。

## Start

> ElasticSearch 需要有java的依赖 ; 由于 某堵高墙的存在 从Docker Hub pull 镜像速度极慢而且容易卡死。于是 我改了下dockerfile,换成了Daocloud Hub的源. 当然 自带梯子的同学可以忽略这段,直接引入Docker Hub的镜像就可以 :FROM java:8

* 从Dockerfile构建镜像: `docker build -t ="dockerfile/elasticsearch" github.com/scbizu/elasticsearch`

* 构建成功后,直接`docker run -d -p 9200:9200 dockerfile/elasticsearch`,就可以在localhost:9200查看到数据了。

## Settings

当然,直接这样跑es是存在风险的,当需要更改es的设置(elasticsearch.yml)的时候,就要进到容器里面改,这样会很不方便 以及会影响服务 所以这是不被提倡的。官方建议的是外挂es的设置和data文件夹,这很容易理解,我们在宿主机上直接更改设置文件和监控es的运行状态显然更合理。通过`docker run `的`-v `标签来进行外挂`Volumn`.

* 在宿主机上创建一个用于外挂的文件夹,比如我们先`mkdir /data`,指定`/data`为外挂文件夹。然后创建目录`log`,`work`,`data`用于监控es的状态,再`touch elasticsearch.yml`来指定es的设置,设置如下:

```

path:
  data:/data/data  #注意下这是要被放到Docker里去的,所以这里挂载的应是docker下的目录,而非宿主机.
  log:/data/log

  plugin:/data/plugin  #可选,如果需要可视化工具的话,需要挂载plugin目录,记得先安装呦~(cd 到{elasticsearch的安装目录}/bin/plugin -install mobz/elasticsearch-head或者lmenezes/elasticsearch-kopf/v2.0.1)
```

## Run it

* 配置好之后 就可以Run起来了

`docker run -d --name es -p 9200:9200 -p 9300:9300 -v /data:/data dockerfile/elasticsearch /elasticsearch/bin/elasticsearch -Des.config=/data/elasticsearch.yml`

* `docker ps`查看下是不是确实跑起来了(访问localhost:9200也可以~)

## Cluster

> 看着黄色的Health毕竟不爽,所以来搭集群吧 ~

ES的分布式集群其实很有趣,可以戳[这里](http://es.xiaoleilu.com/020_Distributed_Cluster/00_Intro.html) 看看究竟是怎么回事。

搭建其实也很简单,在宿主机`/data`下新建`es-master` `es-node1` `es-node2`的文件夹,把之前的`/data/data` `/data/log` `/data/work` `/data/elasticsearch.yml` 都复制一份 扔到前面几个文件夹里面去,如果想要共享配置的话 只需要新建一份就可以了。

然后让`node1` 变为`master`的从节点:

`docker run -d --name es-node1 --link master -v /data/node1:/data dockerfile/elasticsearch /elasticsearch/bin/elasticsearch -Des.config=/data/elasticsearch.yml`

node2同理 ;

之后可以通过Plugin来看看节点状态.如下(选举Agon做主节点了真是尴尬233):

 ![es_head](http://7xqdui.com1.z0.glb.clouddn.com/es.png)

 > 变绿了耶!


## Referer

[Github下的docker elasticsearch repo及其issue](https://github.com/scbizu/elasticsearch)
[饿了么后端小记](https://zhuanlan.zhihu.com/p/20690152)

以上,完.
