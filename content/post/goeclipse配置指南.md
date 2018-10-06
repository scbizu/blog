---
categories:
- 伪技术相关
- golang
tags:
- configuration
- golang
title: goeclipse配置指南
---


> 之前用的是Liteide 但是觉得在Lite里面敲代码一点都不优雅啊，所以准备转Goclipse了 还是Eclipse大法吼啊。。



参考文章：[Goclipse"官方"基佬站md文件](https://github.com/GoClipse/goclipse/blob/latest/documentation/UserGuide.md)

好了,正文开始：

Goclipse=Golang+Eclipse+GDB(debugger)

在配置Goclipse之前 你需要经过一次进化(满足下列condition):





  * Eclipse Mars (or later) -.-就是Eclipse 4.5或更高版本


  * Java VM version 8 or later -。-这个应该不用说了吧。



## 安装开始：



  1. 找安装包:找安装包有两种方式，第一种(翻墙党)直接Eclipse在线安装，第二种通过下载[Goclipse安装包](https://github.com/GoClipse/goclipse.github.io/archive/master.zip)来安装,下下来之后解压->使用release文件夹进行本地安装

  2. 针对第一种安装 ：使用我们熟悉的Eclipse安装方法(help->install new software),然后从这个[site URL](http://goclipse.github.io/releases/)进行Install

  3. 接下来就是漫长的pending时间(取决于梯子的速度咯),pending过后 勾选 Goclipse 的 Item。> 此时CDT的开发环境(C/C++)也会被添加 用来作为Golang的debugger.

  4. 然后  重启Eclipse。。。。。你会发现一个很萌的图标。。



![截图20160121164601](http://7xqdui.com1.z0.glb.clouddn.com/%E6%88%AA%E5%9B%BE20160121164601.png)

## 配置环境:

* 配置GOPATH:

    找到**window->preference**(一些环境的配置选项一般都在这里)

    找到go选项 点它  配置GOROOT(win下Golang的配置不在这篇说明  可以check一下自己的环境变量。)


**注意：Goroot必须与环境变量设置相同，设置完Goroot后 gofmt和godoc会自己生成不用设置。**



* 设置gocode:

  续上步，

  找到go后 下拉它!  你会看到 tools 点它 ！然后 设置下 **gocode** 和 **oracle path** 。

**注：这里的获取gocode和 go oracle path 都是用go get 命令的 ,所以需要装Git版本管理,没有的话需要[下载Git](https://git-scm.com/),并把Git的bin(eg: D:\Git\bin)目录加到环境变量中的Path目录中去。**

![截图20160121223428.png](http://7xqdui.com1.z0.glb.clouddn.com/%E6%88%AA%E5%9B%BE20160121223428.png)


* 新建Go项目:



File->new->Project  ;

Go/Go Project;

*  Go项目构建:

  * 项目目录是某个Gopath的入口文件目录src的子目录，那么这个Project将由这个目录下的package组成

  * 反之,这个目录将被看做一个新的Gopath的入口，并且将由 src,bin,pkg三个子目录组成。需要注意的是这个Gopath只适用于当前Project,并不能全局调用(即被其他Gopath下的Project调用,除非这个Gopath被加入到Global Gopath(全局下的Gopath)


这边废话一下方法(Golang的demo中一般都会提到新Project的创建方式):

(带主入口函数的demo) 切到src目录

  1. 先创建自己的包目录(eg:main)  即在src目录下创建一个文件夹也就是包的意思了。

  2. 再在这个包下创建go file (new->Go file->...)     如下创建:
![截图20160121224429.png](http://7xqdui.com1.z0.glb.clouddn.com/%E6%88%AA%E5%9B%BE20160121224429.png)
  3. 接下来  你就会看到main.go的主入口文件啦  是不是找回了熟悉感了。。。

  ![截图20160122001720.png](http://7xqdui.com1.z0.glb.clouddn.com/%E6%88%AA%E5%9B%BE20160122001720.png)
  
PS:上面那个Build Targets 可以设置Build的方式。

```
    ./...#build   建立所有的当前的包  包括测试包(默认)

    ./...#build-tests 建立当前项目目录下所有的测试包

    ./...#[run-tests] 建立并运行当前项目目录下所有的包
```
