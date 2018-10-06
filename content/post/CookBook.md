---
categories: [伪技术相关]
tags:
- 记录

title: Nace's CookBook
date: 2017-10-07 00:06:00
---

## CookBook

这里会记录一些Nace经常碰到的一些不容易记住的编程相关的 **专业术语** 和 **小Tips** ;

### About Code

> 位运算符

|位运算符|语义/Semantics|e.g.|
|---|---|---|
|&|按位与/AND|0101 & 0011 = 0001|
|I|按位或/OR|0101 I 0011 = 0111|
|^|按位异或/XOR|0101 ^ 0011 = 0110|
|^&|位清除/AND NOT|0110 &^ 1011 = 0100|
|<<|位左移/LEFT SHIFT|0001 << 3 = 1000|
|>>|位右移/Right SHIFT| 1000 >> 3 = 0001|

> TIPS: 位清除(bit clear/AND NOT) 与 异或(XOR) 虽然结果表现的一致 但是 实际表述并不一样。

位移操作的右操作数必须要是`uint`类型
```go
b=:23
x:=1 << b  //panic: (shift count type int,must be unsigned integer)
//Right Code
var t uint
t = 2
x:=1 << t
println(x) // 4
```

### About Arch


### Best Practice in Golang

* 类型转换

在一些高性能场景中,类型转换不能使用type(foo)的形式,可以考虑用unsafe包 从底层存储方式进行转换 会高效很多：
```go
func toString(foo []byte)string{
  return *(*string)(unsafe.Pointer(&foo))
}
//其他转换同理
```

* 构建字符串

字符串相加在数量很多的场景下,性能会变得非常差,因为每次都会给新的string重新分配内存。所以,在需要相加的字符串很多的情况下,用`strings`包的`Join()`或者`bytes`包的`WriteString()/WriteBytes()`,一次性分配好需要分配的内存就会让性能提升很多.
```go
func enlargeString(foo string,boo string)string{
  var buf bytes.Buffer
  buf.WriteString(foo)
  for (i:=0;i<aLargeNumber;i++){
    buf.WriteString(boo)
  }
  return buf.String()
}

```

* 避免Goroutine中的延迟求值

在日常的开发中 我们往往需要在一个LOOP中实现并发 这时候往往这么写:
```go
  //foos is a array slice or else which can be ranged
  for _,foo:=range foos{
    go func(){
      //do sth with foo
    }()
  }
```
但是这样就会遇到一个问题:得到的foo永远都是`index==len(foos)-1`的foo值,这是因为goroutine的延迟求值,这里每次go func的匿名方法并不会被马上执行,当LOOP 执行完的时候才会去执行展开其中的Goroutine.所以正确的写法应该是每次执行`go func`的时候把foo带进去,让goroutine去保存这个状态。具体的代码如下:
```go
  //foos is a array slice or else which can be ranged
  for _,foo:=range foos{
    go func(foo TYPE){
      //do sth with foo
    }(foo)
  }
```

* 关于反射的一些知识

反射是在Runtime层读取对象的内存信息与类型;但是由于没有对象的头指针,无法做到靠其自身来解析对象的类型。依靠`interface{}`来实现反射,`i interface{}`可以保存参数的**类型**和**数据**

> 这里的`interface{}`只是对象的一个复制,并且是不可寻址的,所以,要改变对象的信息只能传入指向此对象的指针

```go
a:=100
reflect.Value(a) //canset:false canaddress:false
reflect.Value(&a) //canset:true canaddress:true
```
利用reflect来导出不可导出类型 任何pkg相对于reflect来说都是外部包。

* Golang中的AOP(面向切面编程)

[什么是AOP?](https://zh.wikipedia.org/wiki/%E9%9D%A2%E5%90%91%E4%BE%A7%E9%9D%A2%E7%9A%84%E7%A8%8B%E5%BA%8F%E8%AE%BE%E8%AE%A1)

在Golang中,反射(reflect)很好地切合了AOP的主题。通过`reflect`在Runtime的时候动态执行方法,当我们用AOP的思想抽象出一个切面之后,也许这个切面是一个日志模块,或是一个审计模块(这些模块具有一个共同点:永远不需要也不可能被需要耦合到业务代码中去)。也可以把这些模块看成一个代码hook，这些模块在不用更改业务代码的同时调用业务代码中的逻辑。一次简单的尝试如下:
```go
  type LogModule struct{}
  func (lg LogModule)logsth()(string,error){
    //log sth ...
  }
  func main(){
    logm：=new(LogModule)
    reflectLog:=reflect.ValueOf(logm)
    m:=reflectLog.MethodByName("logsth")
    in:=[]reflect.Value{}
    //Call logsth in Runtime
    out:=m.Call(in)
    //res
    for _,res:=range out{
      println(res)
    }
  }
```

* 在Golang中使用Set

首先,Golang中是没有`built-in`的`set`关键字的。所以,要实现类似`set`的数据结构,需要使用`map`和`key`来组合成一个`set`。

> 这里的set更多的指的是一个FIFO的map也就是一个ordered-map。

比如,我们现在有一个map:
```go
foo:=make(map[string]string)
foo["go"]="go"
foo["js"]="js"
foo["php"]="php"
```
然后,我们发现遍历这个map的时候,key的顺序有可能是会被更换的:

```go
for key,value:=range foo{
  //key,value的值并不唯一
  println(key,value)
}
```
这是为什么呢?其实仔细想一想就知道,我们为什么要指望`map`来向`set`一样来工作呢?这本来就是两个数据结构啊！所以,我们要的是什么？在这种场景下,我们需要的只是**key按一定顺序排列的map中对应的值**，经过这层思考之后,就可以很快的得出代码：
```go
  //按你想要的顺序遍历map吧..
  keys:=[]string{"go","js","php"}
  for _,key:=range keys{
    println(key，foo[key])
  }
```
再反过来想想 我们为什么希望map按照FIFO顺序来呢?终究是对map的理解出现了误解。
这时候静下来 好好读读[Go blog](https://blog.golang.org/go-maps-in-action)的说明,就会有豁然开朗的感觉了吧：）
