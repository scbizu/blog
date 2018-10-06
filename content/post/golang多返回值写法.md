---
categories:
- 伪技术相关
- golang
tags:
- in action
- golang
title: golang多返回值写法
---

看热闹不嫌事大系列-------->[如何看待函数返回多个值的设计?     From  知乎](https://www.zhihu.com/question/20132013)

这个号称是Golang特性之一的  多返回值写法 给开发者带来了许多便捷的地方，由于Golang的语法中规定没有使用过只是初始化了的变量会报错,所以，引入了多返回值的方法，因为这样做就可以用`_`来舍弃掉之前声明但是又不想使用的变量了。

先来看一段多返回值的标准写法:

```
    package main

    import (
      "fmt"
    )

    func dataOp(data1 float32 ,data2 float32)(float32,float32){

        return data1+data2,data1*data2
        }

    func main() {
       var x float32
       var y float32
        x=1.1
        y=2.2
        added,mutied:=dataOp(x,y)
        fmt.Printf("%f,%f",added,mutied);   
    }
```

顾名思义,也就是dataOp()返回了两个值。(e.g.中是data1+data2和 data1*data2).然后调用的时候的写法就是定义两个变量来接收dataOp的返回值。

然而返回值的先后顺序也必须的一一对应的。

改一下代码，我们舍弃掉第一个返回值。

Change
```
    added,mutied:=dataOp(x,y)
```

To

```
     _,mutied:=dataOp(x,y)
```


验证结果后会发现,返回的是乘积值:2.420000


利用这种原理,我们可以大大简化代码的冗余，而且，我们是可以自由选择返回值,不要的就可以释放掉,优化程序的内存管理。
