---
title: "Go Concurrency Pattern"
date: 2019-12-22T15:07:44+08:00
draft: true
---

# 再谈Go的常用并发模型

> 脱离业务领域谈设计模型都是扯淡，本文旨在抽象几种常用模型，希望在设计业务模型时提供参考

## 前言

转眼已经快写了三年的Go了，因为Go的本身设计的简洁，在平常基本用不到很多设计模式的思想，更别谈并发模型了，看过很多代码，基本都是channel/condition 乱飞，没有文档或者注释辅助的情况下，review/refator 代码简直是噩梦。 为了不让自己成为自己讨厌的人，我决定自己整理下Go常用的并发模式，也正好听Go Time的播客的时候，听JBD提到2016 GopherCon上的「Visualize Go Routine」的topic，又去拜读了相关文章，觉得是时候沉淀下这块知识了。


## 常用并发模型

### Ping Pong

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
