---
categories:
- 伪技术相关
- golang
tags:
- in action
- golang
title: golang bufio实验
---

## Who is he?



好了 开始今天的包

嗯  今天的包的名字是bufio  .

他呢   就是证明golang不单单是脚本语言的证据! 他实现了缓冲 I/O   他包含了io.Reader 和 io.Writer 。





## How TO?



在用之前当然先要New一下咯。

Like this:



    scanner:=bufio.NewScanner(r io.Reader)



这里的r io.Reader是一个io.Reader类型的参数,比如os.Stdin,net.conn啥的 都是可以直接扔进去的。

当然 有io.Reader 怎么能没有io.Writer 说了他们是一对啊(雾

So,Like this:



    writer:=bufio.NewWriter(r io.Writer)



同理   这个io.Writer里面也可以扔stdout了

然后就是从缓存中慢慢输出的过程了

就像这样(默认引入了fmt包):





    fmt.Fprint(writer,"Hello")

    fmt.Fprint(writer,"World")

    //At last Flush it!

    writer.Flush();





啊呀 ，偏了偏了;

Scanner还没写完诶；

不过其实也大同小异；

也不过是遍历 io.Reader嘛 ;

遍历io.Reader需要用到r.Scan()方法;

来看下Scan方法的原型



    func (s *Scanner) Scan() bool



So, Scan方法返回的是boolean值,不加的话 会直接跳过输入(Stdin) 。所以,想要遍历输入的话 一定要加上scanner.Scan()。

所以 我们就来试试呗:



    for scanner.Scan(){
        fmt.Println(scanner.Text())
    }



(s *Scanner).Text()方法就不用说了 就是获取到scanner里面的那一坨,然后把string类型扔回给你;

所以这样就可以实现I/O了(不是脚本哦!)



既然写都写了  那肯定不能忘掉(s *Scanner).Split()咯

Split()好用的地方应该是在可以带一个Split func吧~

比如bufio.ScanWords()就是一个split func  (用来分割输入的  逐字咯)

再如bufio.ScanBytes()也是一个split func()  (返回字节)

还有bufio.ScanLines()也是一个split func() (返回一行<-带有换行标记的行  返回的行不会带有这些标记 所以可能是空行 可以用来做读到末尾的flag)

(不知不觉就队形了~)

喂 不要忘了 转码输出的我啊  bufio.ScanRune()如是喊道  (他可以返回字符的UTF8字符 万恶的中文字?(大雾))



bufio 的 Type的话好像就剩一个bufio.Reader()没有提起了 然而我都不想提>...< 因为跟Writer的用法实际上差不了多少。



所以 就写到这里吧  有什么不对的地方尽管拍砖  预计下一个net包 明天抵达战场(FLAG高高挂起  被歪果仁评论了 感觉好稀奇  虽然我知道只是。。。。。某个莫名其妙的爬虫或者钓鱼网站看上我了(哭

[![gopher_head](http://scnace.cc/wordpress/wp-content/uploads/2016/01/gopher_head-300x161.png)](http://scnace.cc/wordpress/wp-content/uploads/2016/01/gopher_head.png)
