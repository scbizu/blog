---
categories:
- 伪技术相关
- golang
tags:
- in action
- golang
title: golang channel实验
---

信道(channel)是Goroutine之间通信的手段,所以,掌握Channel的用法之于Golang的学习变得尤为重要。



### 信道到底是怎么一种东西?



也许信道与 **UNIX的双向PIPE** 比较类似,也是采用了FIFO的方式来实现调度。由于Goroutine **运行在相同的地址空间** , **所以访问共享内存时必须要做好同步** 。Channel就是为了解决这个问题而出现的。通过他来接收或者发送值。





### Donot BB Show me the code
```
    func main() {

       init:=make(chan int,20)
       init<-1
       init<-2
       fmt.Println(<-init)
       fmt.Println(<-init)
       }
```


这就是Channel的用法了,首先缓冲数据,再取出数据。

By the way,信道的发送和接收方默认是阻塞的,也就是,知道发送方有数据发送过来之前,接收的通道一直都是阻塞的。

当然,这是缓冲Channel的写法。可以缓冲多组数据。

而无缓冲的channel(即 **make(chan TYPE)** )只能用于单组数据的传输,这也是goroutine最常用的方式。


### Goroutine基本运用

用channel来辅助并发(Coroutine) 应该是channel最基础的应用.由于在主线程(main)会在go func()之前跑完,所以导致func()没法跑完  用channel实现的思路是 **用channel去阻塞主线程,直到个go func()全部跑完 再把阻塞取消掉**。在多个Go Routine中,**需要维护一个全局的channel数组,由这个全局channel控制每个Goroutine**。示例代码如下:

```
package main

import "fmt"

func main() {

	fmt.Println("main routine..")
	charr := make([]chan int, 11)
	charr[0] = make(chan int)
//One GoR
	go func(ch chan int) {
		fmt.Println("other routine")
		ch <- 1
	}(charr[0])
//Other GoRs
	for chn := 1; chn < 11; chn++ {
		charr[chn] = make(chan int)
		go func(ch chan int) {
			for i := 0; i < 3; i++ {
				fmt.Println(i)
				ch <- 1
			}
		}(charr[chn])
	}
//解除阻塞..
	for _, ch := range charr {
		<-ch
	}
}

```
稍微对着代码解释下,charr是我设置的全局channel数组  由他负责所有的子Channel  


### 关于Range和Close在channel中的应用



```
    package main

    import (
      "fmt"
    )
    func channel_init(c chan int){

        for i:=0;i<20;i++{
            c <- i
            }
       close(c)
        }
    func main() {

       init:=make(chan int,20)
       channel_init(init)
        for data:=range init{
            fmt.Println(data)
            }
       }
```





这里的range实际是遍历的一种简写方法。作用也是遍历channel里的所有数据。range给我们带来了很大的便利

而close(),便是关闭当前信道。**注意!close一定要在生产的地方去调用,在消费的地方调用会引起panic**.channel作为一种缓存类型,并不需要经常去关闭,只要在确实没有数据发的时候关闭信道就可以了。





### 升阶--让我们在一起吧



思想:在遇到非常复杂的计算时,我们可以分多路对此计算进行分割计算，之后再把三路的计算结果放入总信道，然后从总信道里取值，以此来缩短程序的运行时间。

The Code is here:


```
    package main

    import (
      "fmt"
      "time"
      "math/rand"
    )

    func simu_cacul(x int) int{
        time.Sleep(time.Duration(rand.Intn(10)) * time.Millisecond)
        return 50-x
        }

    func branch(data int) chan int{
        //构造branch
        branch:=make(chan int)
        //匿名函数表达式执行
        go func(){
            //部分计算过程 放入这个分支
            branch <- simu_cacul(data)
            }()
        return branch
        }
    //合并全部分支
    func merge(aChan ...chan int)chan int{
        //aChan是一个Slice类型
        result:=make(chan int)
        for _,v:=range aChan{
            //merge    取出传入信道的数据传到result信道
            go func(c chan int){result <- <-c}(v)
            }
        return result;
        }

    func main() {
        res:=merge(branch(1),branch(2),branch(3))
        for i:=0;i<3;i++{
            fmt.Println(<-res)
            }
       }
```




(嘘！这里我们假装50-x是一个非常耗时的计算。。。用了Sleep来强行伪装。。。。)





### 监视器



Golang中黑科技炒鸡多的,这就是我喜欢它的原因吧~~比如这个SELECT是用来监测信道上的数据流动

select默认是阻塞的,只有当监听的信道接收或发送时才可以运行,然后当有多个信道满足条件时,select会选择一个执行。(感觉用法跟Switch‘有点像   然而只是语法有点像 case default啥的

然后上面的代码就可以开始简化了:
```
    //合并全部分支
    func merge(aChan ...chan int)chan int{
        //aChan是一个Slice类型
        result:=make(chan int)
        go func(){
            for _,v:=range aChan{
                select{
                        //监听从v信道流出
                    case c:=<-v:
                       result <- c
                    }
                }
            }()
        return result;
        }
```


SELECT 还有一个 **设置超时** 功能,用来避免全部阻塞的情况



    case <-time.After(5*time.Second):
    //do sth here
