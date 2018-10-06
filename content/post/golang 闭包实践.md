---
categories:
- 伪技术相关
- golang
tags:
- in action
- golang
title: golang 闭包实践
---

## Problem



前几天在 [知乎](https://www.zhihu.com/question/34210214)  上看到了一个很好的问题

回答中也有很多干货

于是 我也想加深一下对闭包的理解

于是  写下了这篇部落格。



## Description

  * **作用域(scopes)**

作用域,它的名字就是对它很好的解释,闭包使得**内部函数可以访问外部的变量。**

这种作用域往往就是用(匿名)函数来实现的。

所以 从理解闭包来说  把闭包理解为 **有(记忆)状态的函数块** 比较容易。就像MDN上(对于JavaScript上)写的一样:



> Closures are functions that refer to independent (free) variables. In other words, the function defined in the closure 'remembers' the environment in which it was created.





我们可以利用这个函数(作用域)对一些外部变量进行操作 从而达到我们的目的。


  * **把函数作为值传递(First-Class)**

在闭包中,我们把 **函数视为头等公民(first class)** ,把函数当做变量来传递;我们在函数内部随便玩,以达到我们的目的 (比如我们喜欢把函数值赋给变量啦,还是直接return这个函数啦什么的)这么做的好处就是 **打破了原本程序的封闭性,原本的程序就是外部Scope归外部,函数内部的Scope归内部,函数内部的变量,方法什么的都是存在栈(Stack)上的,随着这个的scope的结束而被GC拖走** 。现在就是在两个环境间开了扇传送门,在某个变量快要被GC拖走的时候 我们把他通过闭包保留下来 给他造个分身留下来(有OOP基础的看到这里 也许会将它与this.xxx比较  这个我接下来说~)



当然,匿名函数与闭包才是最棒的CP~ 可以直接在主代码里给扔个func什么的最棒了(如何编写不可维护的代码即视感2333)





## Some Codes

```
//written in Golang

//Created By scance

    package main
    //Golang的闭包实验
    import (
    "fmt"
    )
    //闭包写法  返回两个函数
    func closure(num int)(func(),func(int)){
        //匿名函数
        first:=func(){
            num++
            fmt.Println(num)    
            }
        //匿名函数
        second:=func(xnum int){
            xnum++
            fmt.Println(xnum)
            }
        //返回函数
        return first,second
        }
    func main() {
      //Golang的双返回值写法
      first,second:=closure(10)
      first() //11
     //假装要把12存起来
      second(12) //13
      second(12) //13
    }
```

```
    package main
    //Golang的闭包实验
    import (
    "fmt"
    )
    //第二个闭包
    func second_closure() (func()){
        //局部变量count
        count:=0

        func1:=func(){
            count++
            fmt.Println(count)
            //log.Fatal(count)      
            }
        //匿名函数返回
        return func1
        }
    func main() {
        //函数返回值
    //      func1:=second_closure()
    //      func1()  //1
    //      func1()   //2
    }

```


```
    package main
    //Golang的闭包实验
    import (
    "fmt"
    )
    func main() {
        //第三个匿名函数
       func2:=func()(func()){
            count2:=0

            return func(){
                count2++
                fmt.Println(count2)
                }
            }()

       func2() //1
       func2()  //2
    }

```





## Extension





### topic 真正的对象 vs 闭包

在OOP编程中 我往往会倾向于  用this.xxx来表示一个变量 使它跟这个Class一起共存亡。

但是在函数为主的语言中(JavaScript,Golang等)   我觉得用闭包来实现这种 **switch scope** 不是显得更优(zhuang)雅(bi)嘛 !少写了一个Class不说(主要是懒),可以让这个外部变量打入这个函数式的内部环境,不是显得更方便嘛!




所以  其实 **从实现的效果来说没什么区别** (gc方面其实还有区别),但是用工具而已,纠结太太太深也不好，所以，就当做没区别好了(逃


**可能有些地方BB的不怎么准确  我会在接下来更深入的学习中改的(拜**

## THE END



离散数学老师: 所以你在说什么?

[离散数学中的闭包集合](https://zh.m.wikipedia.org/wiki/%E9%97%AD%E5%8C%85_(%E6%95%B0%E5%AD%A6)
