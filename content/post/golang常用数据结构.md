---
categories:
- 伪技术相关
- golang
tags:
- in action
- golang
title: golang常用数据结构
---

> 原post:[RUSS Cox Blog Go Data Structure](http://research.swtch.com/godata)



## 一些常用数据格式如下所示:



![godata](http://scnace.cc/wordpress/wp-content/uploads/2016/03/godata.png)

图解：

i在memory中表示一个32 bits的数据,也就是4 bytes的数据。也就是说,int 的 底层也是存了4 bytes(32位机)

j也存了4 bytes的数据,也就是说,int32 格式的也是 4 bytes

f也存了4 bytes的数据,也就是说说float32在底层也是4bytes

而 bytes 根据它指定的bytes来赋予内存,(e.g. [5]bytes 就是 5 bytes)

表示Array时 是它里面源数据格式的叠加(e.g. [4]int 也就是4个int格式的叠加 也就是4*4=16 bytes)





## 结构体 和 指针 (Struct and pointer)



Go和C语言一样,让开发者自己去掌控指针,而不仅仅局限于使用提供的指针。

下面以 Struct 为例,首先定义一个结构体:



    type Point struct {X,Y int}



[![截图20160304190938](http://scnace.cc/wordpress/wp-content/uploads/2016/03/截图20160304190938-300x79.png)](http://scnace.cc/wordpress/wp-content/uploads/2016/03/截图20160304190938.png)



如上:

Point{10,20}两个int数据 加起来一共有8Bytes 是直接存在内存当中的;

而 &Point{10,20}定义了 指向这个Point的指针,所以只有4 bytes,指针指向后面那个Point的首地址。

再如:





    type Rect1 struct {min,max Point}
    type Rect2 strcut {min,max *Point}



[![截图20160304191715](http://scnace.cc/wordpress/wp-content/uploads/2016/03/截图20160304191715-300x38.png)](http://scnace.cc/wordpress/wp-content/uploads/2016/03/截图20160304191715.png)

前者表示四个Point格式的数据,一共有16 bytes;

后者其实只有两个Point类型的数据,每个field都会指向真正的数据的首地址,所以,加起来只有8 Bytes;





## 字符串(Strings)



[![截图20160304192549](http://scnace.cc/wordpress/wp-content/uploads/2016/03/截图20160304192549-300x94.png)](http://scnace.cc/wordpress/wp-content/uploads/2016/03/截图20160304192549.png)

(灰色箭头表示在运行中是这样的,但在程序中不会写出来。)

Golang中的String数据结构 包含一个 **指针** 以及 **一个长度位**。指针位为 4 bytes 长度位也为 4 bytes,加起来是 8 bytes,其中,指针指向字符串的首地址。由于字符串(String)都是一成不变(immutable)的,而且许多字符串共享一个空间是安全的,所以，切片操作时用到的新的字符串结构(包括新的指针和长度位)实际上就是指向源数据。也就是说,在完成切片操作时 不需要进行再次分配与拷贝,从而使得切片操作更有效率地去获取准确的索引。





## 切片(Slices)



Slice的数据结构如下所示:

[![截图20160304195102](http://scnace.cc/wordpress/wp-content/uploads/2016/03/截图20160304195102-300x63.png)](http://scnace.cc/wordpress/wp-content/uploads/2016/03/截图20160304195102.png)

切片操作是对数组(Array)的便捷操作,在日常开发中经常会看到它的身影(<del>卧槽,这reference难道真的要翻译成参考?</del>)

由上图可以看到:

Slice的结构为3个4-bytes 结构 。第一个 4-bytes位是用来存放指向 源数据 的指针的, 第二个 4-bytes位是用来存放字符串长度的,第三个 4-bytes结构是描述数组容量的。

长度:索引操作时的上限;如x[i]  此处即为i的上限

容量:切片操作时的上限;如x[i:j]

就像对字符串切片一样,对数组切片也不会对源进行拷贝:只是新建了一个新的带 指针 长度 容量 的结构。

在例子中我们可以看到在切片操作时的x 并没有复制这个数组 而是新建了一个结构。

当然,例子中的这个新建的y的长度为2,y[0]和y[1]是空的索引值,但是容量为4.y[0:4]就是空值啦~

因为 Slice是多字结构,不是指针,不用去分配内存,而且slice头也可以一直保存在栈中。这种实现方法比C语言中的通过精确指针和长度匹配的方法更简便。Go本来就是这么做的,但是发现这样做意味着每个Slice操作都要分配新的内存给它,即便是在最快的分配情况下,这种做法也给GC(垃圾收集器,Garbage Collection)带来了不小的负担,并且我们还发现,在进行字符串操作时避免使用切片有利于提高程序的性能。





## 新建数据格式 (New AND Make)



[![截图20160304225726](http://scnace.cc/wordpress/wp-content/uploads/2016/03/截图20160304225726-295x300.png)](http://scnace.cc/wordpress/wp-content/uploads/2016/03/截图20160304225726.png)

new() 返回的是 一个指向归零的内存的指针  make() 返回的是 一个复杂的结构，不是指针。

但是由于C/C++的传统new(Point)的形式不被大家提倡,而都改用make(* Point),然而,在实验中发现,这种方法的效果却不如预期的好。



(TO BE CONTINUED.....)
