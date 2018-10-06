---
categories:
- 伪技术相关
tags:
- 日常积累

title: JSON的语义化挑战--Collection+JSON
thumbnail: https://ws1.sinaimg.cn/large/005uxZXHly1fg9ipu78t9j322w0uw42q.jpg
---

## 背景
在长达数十年的API表述格式的竞争中,XML终究还是败下阵来,JSON因为他的简洁以及对JavaScript的友好,被大肆地铺开使用。然而,作为一种单纯的单纯的文本表述格式,JSON是缺少严格的语义的,定义比较随意的JSON可能造成交互上的困惑,如果有一种标准化的JSON格式,并且让大家都遵守这个标准,那么肯定会节省很多时间和多余的理解JSON语义的工作。在`《RESTFUL WEB APIs》`的安利下,我决定尝试下[Collection+JSON](http://amundsen.com/media-types/collection/format/),这个号称 **最有希望的JSON的fiat标准**。

## Collection+JSON的语义化

### “集合”

就像名字一样,Collection+JSON的设计中到处都在体现"集合模式"的概念。

一个集合可以看成是一组资源的聚合,一个集合中会包含指向这些资源的链接和对这些资源的描述。从这也可以看出,这个标准也符合REST的设计规范。

> 集合: 以链接方式罗列其他资源的资源。

### 具体定义

先给出一个Request和Response的具体描述:

* Request
```
GET /simpleCollection/ HTTP/1.1
Host: blog.scnace.cc
Accept: application/vnd.collection+json
```

* Response
```
//Response Header
200 OK HTTP/1.1
Content-Type: application/vnd.collection+json
Content-Length: xxx
```
* Document
```JSON
{"collection":{
  "version":"1.0",
  "href": "https://blog.scnace.cc/api/",
  "items":[{
        "href": "/api/post/1",
        "data": [
          {"name":"text","value":"Hi nace :)"},
          {"name":"other name","value":"other value"}
        ],
    }],
  "links":[
    {"href":"/logo.png","rel":"icon"}
  ],
  "queries":[{
    "href":"/api/search",
    "rel":"search",
    "prompt": "Lets search sth.",
    "data":[
        {"name":"text","value":"nace"}
      ]
    }],
    "template":{
      "data":[
        {"name":"text","value":"Hi nace,nice to meet you:)","prompt":"New Text Message"}
      ]
    }
  }
}
```

如上就是一个标准的Collection+JSON的文档,它将包含如下字段:

* href : 指向集合本身的永久链接(Response Body);
* items: 包含指向集合成员的链接,以及他们的部分表述(Response Body);
* links: 指向其他与此集合相关资源的链接;可能是一张图片 或是 一段视频(Response Body);
* queries: 用于搜索此集合的控件(Request Body);
* template: 用于向集合添加新的子集;

### CRUD
  Collection+JSON标准下的增删改查同样通过`RESTful`接口来实现:
* Create:在集合中插入新信息通过 **写入模板** 实现

```
//Request Header

POST /simpleCollection/ HTTP/1.1
Host: blog.scnace.cc
Accept: application/vnd.collection+json
```
```JSON
//Request Body
{
  "template":{
    "data":[
      {"name":"text","value":"Hi nace,nice to meet you:)","prompt":"New Text Message"}
    ]
  }
}
```
> name,value是一对k,v结构的数据;prompt是对这条数据的说明或者注释。

* Read:在集合中搜索信息通过 **搜索控件** 实现

```
//Request Header

POST /simpleCollection/ HTTP/1.1
Host: blog.scnace.cc
Accept: application/vnd.collection+json
```
```JSON
//Request Body
{
  "queries":[{
    "href":"/api/search",
    "rel":"search",
    "prompt": "Lets search sth.",
    "data":[
        {"name":"start","value":"na"},
        {"name":"end","value":"ce"}
      ]
    }]
}
```
* Update:向集合中的子项`PUT` **写入模板** 来修改子项的数据

```
//Request Header

PUT /api/post/1 HTTP/1.1
Host: blog.scnace.cc
Accept: application/vnd.collection+json
```
```JSON
{
  "template":{
    "data":[
      {"name":"text","value":"Hi bizu:)","prompt":"New Text Message"}
    ]
  }
}
```
* Delete:向集合中的子项发送`DELETE`来删除子项的数据

```
//Request Header

DELETE /api/post/1 HTTP/1.1
Host: blog.scnace.cc
Accept: application/vnd.collection+json
```

### 分页

在处理较多子集的场景中,**分页** 是标准中不可或缺的一环,但是,Collection+JSON并没有显示地处理这个问题。办法总是会有的,分页这种平常的需求也是肯定要满足的,于是,我们可以采用关联其他媒体资源的方法实现分页:
```JSON
{
  "links":[{
    "name":"next_page",
    "rel":"next",
    "href":"/api/post/2",
    "render": "link",
    "prompt":"Next"
  },{
    "name":"previous_page",
    "rel":"previous",
    "href":"/api/post/pre",
    "render": "link",
    "prompt":"Prev"
  }
  ]
}
```

### 实现

> 这个标准工具包已经在我Side-Project的TODO List里面了233

## Reference

[个人主页的Collection+JSON说明](http://amundsen.com/media-types/collection/format/)

[GitHub](https://github.com/collection-json)
