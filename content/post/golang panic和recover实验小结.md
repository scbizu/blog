---
categories:
- 伪技术相关
- golang
tags:
- in action
- golang
title: golang panic和recover实验小结
---
## 认识Panic和Recover





### Panic



这是Golang中的一个内建函数，可以中断原有的控制流程，进入一个令人恐慌的流程中。当函数F调用Panic，函数F的执行将被中断，但是F中的延迟函数会正常执行,然后F返回到调用它的地方。这个恐慌可以直接调用panic产生，也可以由运行时的错误产生。





### Recover



Golang的内建函数。可以让进入恐慌的goroutine恢复过来。并且**Recover**只在延迟函数中有效，在正常情况下goroutine会返回Nil 但是一旦陷入恐慌,Recover就会返回当前的陷入恐慌的panic信息,并恢复程序的正常执行。





## DEMO





    package main

    import (
      "fmt"
    )

    func ifpanic(){
        //recover

          //unnamed function
          defer func(){
            if r:=recover();r!=nil{
                fmt.Println("recover in ifpanic()",r);
                }
            }()
          //Catch panic
          fmt.Println("Calling panic block");
          panicblock(0);
          fmt.Println("Returning from panicblock normally");
        }

    func panicblock(data int){
        //creat a panic manually by recursion
        if data>=3 {
            fmt.Println("panic ing....");
            panic(fmt.Sprintf("%v",data));
            }
         defer fmt.Println("defer block",data)
         fmt.Println("Printing now",data);
         panicblock(data+1)
        }


    // func dataOp(data1 float32 ,data2 float32)(float32,float32){
    //  
    //  return data1+data2,data1*data2
    //  }
    //只有一个defer 的情景
    func main() {
        ifpanic();
        fmt.Printf("Return normally ...");  
    }




输出为:



    Calling panic block
    Printing now 0
    Printing now 1
    Printing now 2
    panic ing....
    defer block 2
    defer block 1
    defer block 0
    recover in ifpanic() 3
    Return normally ...



有趣的是  如果我们去掉Ifpanic()里面的defer关键字(让其马上执行,不延迟)

即 defer func(){}()  --->  func(){}()

输出马上就会变为:



    Calling panic block
    Printing now 0
    Printing now 1
    Printing now 2
    panic ing....
    defer block 2
    defer block 1
    defer block 0
    panic: 3

    goroutine 1 [running]:
    main.panicblock(0x3)
        C:/Users/A450V/workspace/godemo/src/main/main.go:26 +0x29d
    main.panicblock(0x2)
        C:/Users/A450V/workspace/godemo/src/main/main.go:30 +0x617
    main.panicblock(0x1)
        C:/Users/A450V/workspace/godemo/src/main/main.go:30 +0x617
    main.panicblock(0x0)
        C:/Users/A450V/workspace/godemo/src/main/main.go:30 +0x617
    main.ifpanic()
        C:/Users/A450V/workspace/godemo/src/main/main.go:18 +0x123
    main.main()
        C:/Users/A450V/workspace/godemo/src/main/main.go:40 +0x1f




(注:GOPATH 为C:/Users/A450V/workspace/godemo/ )

当异常的JSON数据格式被捕获到,就会优先(first-level)调用PANIC去减轻栈的压力,然后会返回一个error code 并自动调用recover返回程序。。

由此  证实了如定义所说。
