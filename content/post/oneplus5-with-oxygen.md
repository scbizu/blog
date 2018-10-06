---
categories:
- 伪极客
title: 用adb+fastboot升级一加5到氧OS
date: 2017-06-30 02:15:10
tags:
- 搞机日常
- 作死
thumbnail: https://ws1.sinaimg.cn/large/005uxZXHgy1fh4pcqeq5xj30zk0k0td8.jpg
---
## 舍弃H2OS
虽然,H2OS已经大幅接近原生,但是毕竟是针对国内大环境定制化的系统,刚拿到手看到预装的 **百毒输入法OnePlus定制版** 就有一种强烈的反感,一上手就装上了Google全套,终于用着爽了一点,但是有时候打开Google Now的应用列表,看着不能卸的预装应用总会觉得挺变扭的。

终于,由于不知道是什么APP经常往我桌面上扔莫名其妙的icon,终于忍无可忍开始动手刷氧OS。本来刚上手就想刷氧OS的，可是那时候一来工作上的事比较忙,二来一加官方没有出最新的Oxygen的ROM,所以只好先放着。

好奇桌面为啥会 **出现莫名其妙的icon** 的同学可以看下这个[帖子](https://www.v2ex.com/t/371720),好像是通过在APP管理里面停用默认浏览器就可以fix了。

## Hello OxygenOS

### 准备(Preparing)

先列一下需要准备的工具(由于博主是macOS下刷的,所以工具会比较通用):

* 推荐用 **类似Google Drive** 的东西做好当前数据的备份,然后对手机进行 **双清** 操作.
* [platform sdk](https://developer.android.com/studio/releases/platform-tools.html):其中会包含adb shell 和 fastboot 等一系列工具
* [TWRP for OnePlus5](https://forum.xda-developers.com/oneplus-5/development/twrp-oneplus-5-wip-t3626134): 也可以直接在[这里(自备梯子)](https://drive.google.com/file/d/0B0pr-ZA5b-1pdmdLNDFoTWtUX1k/view) 或者 [这里(不需要梯子)](https://pan.baidu.com/s/1bpxomJH)下载,[官方](http://downloads.oneplus.net/oneplus-5/oneplus_5_oxygenos_4.5.3/)现在也提供了一加5 氧OS的TWEP Rec(拉到页面最下面会看到 **Download Oxygen Recovey (Optional)** 字样).
* [Oxygen 4.5.3](http://downloads.oneplus.net/oneplus-5/oneplus_5_oxygenos_4.5.3/):官方的ROM包,较大,推荐使用做过下载优化的下载工具下载
* [文件传输工具](https://www.android.com/filetransfer/):可以参考Google官方的[FAQ](https://support.google.com/nexus/answer/2840804)

### 使用adb和fastboot解锁Bootloader(Unlock your device)

> 进入这里之前,请确保已经正确下载好了[platform sdk](https://developer.android.com/studio/releases/platform-tools.html).

那么插入USB数据线,开搞!

打开手机的 **开发者模式**(传说中的点击7下版本号大法),BTW,点击安卓系统版本号(如果是Android 7+的话)会有捉猫游戏的彩蛋(逃

打开开发者模式里面的USB Debuging，然后

```
# cd 到 platform-tools对应的文件夹
# 首先检查下USB连接是否正常
$ ./adb devices
# 正常情况下 应该会输出一个序列号.

```

打开 **开发者模式 -> OEM 解锁**,之后 启用 **高级重启**;

接下来有两种方法进入bootloader:

命令行(adb):

```
# cd 到 platform-tools对应的文件夹
$ ./adb reboot bootloader
```

手动模式: 长按电源键 关闭手机；完全关闭之后,长按 电源键 和 音量`↓`键 选择进入bootloader。


进入bootloader;

```
# cd 到 platform-tools对应的文件夹
# 确保USB连接正常
$ ./fastboot devices
# 正常情况下 应该会输出一个序列号.

# 解锁bootloader
$ ./fastboot oem unlock
```
不出意外的话 手机这时又会进入重启 继续进入fastboot模式。

### 刷入TWRP Rec(Flash TWRP)
> 进入这步之前 请确保你已经准备好了 [TWRP for OnePlus5](https://forum.xda-developers.com/oneplus-5/development/twrp-oneplus-5-wip-t3626134)

```
# cd 到 platform-tools对应的文件夹
# 准备刷入TWRP REC;刷其他型号的TWRP Rec会报错;
$ ./fastboot flash recovery $where_you_store_your_rec #$where_you_store_your_rec需要换成你TWRP下载到本地的位置
#进入TWRP recovery
$ ./fastboot boot $where_you_store_your_rec
#$where_you_store_your_rec需要换成你TWRP下载到本地的位置
```

这样手机上就进入TWRP了,选择对System可写之后(即移动滑块到最右端,怕麻烦还可以勾选 **Nerve show this screen during boot again**),就可以选择TWRP对系统的操作了。

### 选择OTA升级包(Choose your oxygen ROM)
> 进入这步之前请确保你已经下载了[Oxygen 4.5.3](http://downloads.oneplus.net/oneplus-5/oneplus_5_oxygenos_4.5.3/),并且已经通过 **文件传输工具** 把文件传到了手机的文件系统中

选择`Install(安装)`标签,找到在手机文件系统中存在的Oxygen的ROM,勾选 检验文件,然后滑块滑到最右侧开始刷入Oxygen ORM。

等待一会后 就可以让手机Reboot,进入OxygenOS啦！

刷机之后 推荐把bootloader重新锁起来.

```
# cd 到 platform-tools对应的文件夹
$ ./fastboot oem lock
```

## 使用体验

作为Google服务深度依赖者/爱好折(zuo)腾(si)的geeker,还是极度推荐刷氧OS的,可以极大力度的避开内置流氓软件的骚扰和套路。当然,使用氧OS的前提就是你要有一把好的梯子。。

(APP列表清净的感觉实在是太美好了! 终于能体验到真*原生了！坐等Android 8 了~)

之后 应该还会更新更多的使(zuo)用(si)体(jing)验(li)  （的吧）。

以上。

## References
* [oneplus](http://downloads.oneplus.net/oneplus-5/oneplus_5_oxygenos_4.5.3/)
* [一加3 Root教程](https://www.chiphell.com/thread-1695757-1-1.html)
