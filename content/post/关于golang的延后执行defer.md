---
categories:
- 伪技术相关
- golang
tags:
- in action
- golang
title: 关于golang的延后执行defer
---

> defer:在函数执行到最后时,再进行调用声明了defer的函数。</blockquote>



DEMO：

```
    package main

    import (
      "fmt"
    )

    func defer_first(){
          fmt.Println("Hi ! I'm first defer");
        }



     func dataOp(data1 float32 ,data2 float32)(float32,float32){

        return data1+data2,data1*data2
        }
    //只有一个defer 的情景
    func main() {
       var x float32
       var y float32
        x=1.1
        y=2.2
        defer defer_first()
        _,mutied:=dataOp(x,y)
        fmt.Printf("%f\n",mutied);  
    }
```



以上demo输出为:



    2.420000
    Hi ! I'm first defer



以上demo可以看出defer是最后才会调用的,也就是在函数main退出前调用的。

所以,这种场景最实用的地方应该是 在 文件I/O里面  如果每次文件Close操作都需要关闭相应的资源,次数一多,极有可能造成**资源泄露**的问题。调用了defer之后只要在一个函数结束前去延时调用资源Close函数就可以了。



第二个Demo


```
    package main

    import (
      "fmt"
    )

    func defer_first(){
          fmt.Println("Hi ! I'm first defer");
        }

    func defer_second(){
        fmt.Println("Hi! I'am second defer")
        }


     func dataOp(data1 float32 ,data2 float32)(float32,float32){

        return data1+data2,data1*data2
        }
    //两个defer 的情景
    func main() {
       var x float32
       var y float32
        x=1.1
        y=2.2
        defer defer_first()
        defer defer_second()    
        _,mutied:=dataOp(x,y)
        fmt.Printf("%f\n",mutied);  
    }

```


输出为:


```
    2.420000
    Hi! I'am second defer
    Hi ! I'm first defer

```


由以上demo可知:Golang 的defer操作时采用后进先出(Stack?)模式的，后定义的second defer 反而在first之前输出了。





以上   关于  Golang defer 的实验。
