---
title: "Go Concurrency Pattern"
date: 2019-12-22T15:07:44+08:00
draft: false
---

# 再谈Go的常用并发模型

> 脱离业务领域谈设计模型都是扯淡，本文旨在抽象几种常用模型，希望在设计业务模型时提供参考

## 前言

转眼已经快写了三年的Go了，因为Go的本身设计的简洁，在平常基本用不到很多设计模式的思想，更别谈并发模型了，看过很多代码，基本都是channel/condition 乱飞，没有文档或者注释辅助的情况下，review/refator 代码简直是噩梦。 为了不让自己成为自己讨厌的人，我决定自己整理下Go常用的并发模式，也正好听Go Time的播客的时候，听JBD提到2016 GopherCon上的「Visualize Go Routine」的topic，又去拜读了相关文章，觉得是时候沉淀下这块知识了。


## 常用并发模型

### 引子: Timer

```go
package main

import "time"

func timer(d time.Duration) <-chan int {
    c := make(chan int)
    go func() {
        time.Sleep(d)
        c <- 1
    }()
    return c
}

func main() {
    // 开启N个定时器，但是要注意这N个定时器并不是并发执行的。
    for i := 0; i < N; i++ {
        c := timer(1 * time.Second)
        <-c
    }
}
```

这是一个很简单的并发模型的引子，目的是构造一个**定时器**，或者构造一个**延时任务** 。

(为什么这边不用timer.Ticker ？ 因为作为一个引子，这个模型会引导我们的思想尽量往并发模型上靠拢。)

### Ping Pong

> 让数据在Go Routine间反复横跳 !

```go
package main

import (
	"fmt"
	"time"
)

type Ball struct {
	hits int
}

func main() {

	table := make(chan *Ball)
	go player(1,table)
	go player(2,table)

	// 比赛开始
	table <- &Ball{}
	// 开始一场1s的比赛吧！
	time.Sleep(1 * time.Second)
	// 比赛结束
	fmt.Printf("total hits: %d\n", (<-table).hits)
}

func player(pnumber int,table chan *Ball) {
	for {
        ball := <-table
        fmt.Printf("player %d start to hit the ball",pnumber)
		ball.hits++
		// ping or pong
		time.Sleep(100 * time.Millisecond)
		table <- ball
	}
}
```

> 在上面代码实现中，我们假设有两个玩家，每场比赛1s，玩家击球时间100ms。

这个是有名的[乒乓模型](https://talks.golang.org/2013/advconc.slide#6)。在这个模型中，球(数据)会在N个玩家(Go Routine所持有的channel)中传递，直到到达指定时间，球(数据)被回收。

这个模式还有一个tips可以用来扩展：因为Go Runtime维护了一个channel的Receiver Queue（队列，具有FIFO的特性），所以导致总是最后一个Go Routine所在的channel可以先得到数据，并且确保了每次都可以固定顺序传递，所以不会导致同一个channel执行多次的情况。也即：按照代码的定义顺序，我们有chan recv1,chan recv2,chan recv3,那么receiver拿到数据的顺序一定是`3->2->1->3`.

### Fan-In

> 从多个数据源收集数据并把这些数据整合到同一个channel,也就是All for one 模式 (果然这么叫好奇怪

```go
package main

import (
    "fmt"
    "time"
)

func producer(ch chan int, d time.Duration) {
    var i int
    for {
        ch <- i
        i++
        time.Sleep(d)
    }
}

func reader(out chan int) {
    for x := range out {
        fmt.Println(x)
    }
}

func main() {
    // producer的集合channel
    ch := make(chan int)
    // reader channel
    out := make(chan int)
    go producer(ch, 100*time.Millisecond)
    go producer(ch, 250*time.Millisecond)
    go reader(out)
    for i := range ch {
        out <- i
    }
}
```

### Fan-Out

> 从一个channel中读取数据，分配到多个channel里面进行处理，也就是one for all 模式。

也可以说是 workerpool 模式。

```go
package main

import (
    "fmt"
    "sync"
    "time"
)

func worker(tasksCh <-chan int, wg *sync.WaitGroup) {
    defer wg.Done()
    for {
        task, ok := <-tasksCh
        if !ok {
            return
        }
        d := time.Duration(task) * time.Millisecond
        time.Sleep(d)
        fmt.Println("processing task", task)
    }
}

func pool(wg *sync.WaitGroup, workers, tasks int) {
    tasksCh := make(chan int)

    for i := 0; i < workers; i++ {
        // N个worker
        go worker(tasksCh, wg)
    }

    for i := 0; i < tasks; i++ {
        // 把task放入task chan
        tasksCh <- i
    }

    close(tasksCh)
}

func main() {
    var wg sync.WaitGroup
    wg.Add(36)
    go pool(&wg, 36, 50)
    wg.Wait()
}
```

当然，这种方式还可以扩展成为 subworker 模式，由worker的channel再把task 分发出去。（当然还能是sub subworker 啥的


### Daisy Chain

> refer: https://talks.golang.org/2012/concurrency.slide#39

```go
//  Go Routine 一号
func f(left, right chan int) {
    left <- 1 + <-right
}

func main() {
    const n = 10000
    leftmost := make(chan int)
    // right 是`leftmost`的一个引用，他们指向同一个内存地址
    right := leftmost
    // left 也是`leftmost`的一个引用
    left := leftmost
    for i := 0; i < n; i++ {
        // 为什么在这里要重新定义`right` ？
        // 如果这里不重新定义`right`(把`right`指向新的地址)，在「Go Routine 二号」这里就会直接往这个channel塞1了
        // 然后因为right和leftmost都指向同一个地址，最后就直接输出1退出了，甚至都没「Go Routine 一号」什么事了
        // 这里还需要注意，Go Routine的调用栈是FILO（先入后出）的
        right = make(chan int)
        // 调用「Go Routine 一号」，目的是使`<-left`增1
        // 这里特别需要注意index == 0 的情况 ，此时，`left`是指向`leftmost`,也只有此时，`left`是指向`leftmost`的，
        // 这也保证了，「Go Routine 一号」最后一次的出栈可以让值正确地赋到`leftmost`上.
        go f(left, right)
        // 把`right`赋值给`left`，这里非常关键，这里除了代码表面上的保留`right`之前的状态，还覆盖了`left`的地址，
        // 让其之后不再指向`leftmost`的地址，而是指向`right`的地址，以形成关键的`daisy chain`,
        // daisy chain(展开): left(应该是leftmost) <-1 + ... + 1 + 1 + (1 + <- right .)
        left = right
    }

    //  Go Routine 二号
    // 这个 匿名Go Routine 必须放在for循环之后，确保传入这个go routine的是最后一个`right`, 原因可以结合上面的展开daisy chain 看，那么如果把这个匿名 go routine放在循环之前会发生什么呢？
    go func(c chan int) { c <- 1 }(right)
    fmt.Println(<-leftmost)
}
```
这段代码很有意思，也非常巧妙，在[SOF](https://stackoverflow.com/questions/26135616/understand-the-code-go-concurrency-pattern-daisy-chain)上也有相关讨论， 任何一个地方动一下，可能最后的结果都可能不一样。

使用场景： Daisy Chain的模式，在编写链式workerflow时显得非常高效，假设一个job需要被多个handler顺序处理，那么这种模式在有非常多(成千上万)个handler时，会有奇效。

[这里](https://github.com/massimo-marino/workers-daisy-chain)有人也用Daisy Chain实现了一个类似workpool的东西，对这种模式实际应用场景感兴趣的同学也可以参考下。

### 总结

并发模式不是银弹，如果串行的模式能花更少的代价造出符合当前场景的模型，那么, 何尝又不是不可呢？

先就酱，这又是一个长期坑， To Be Continued...
