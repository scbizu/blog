---
categories:
- 伪技术相关
tags:
- 日常积累

title: ShadowSocksX查看已经储存的Server信息
---

我什么都不知道,就单纯记录下之后可能用的到的命令行:

```shell
ʕ ◔ ϖ ◔ ʔ   ➜  ~ cd ~/Library
ʕ ◔ ϖ ◔ ʔ   ➜  Library cd Preferences
ʕ ◔ ϖ ◔ ʔ   ➜  Preferences cp ~/Library/Preferences/clowwindy.ShadowsocksX.plist ~/backups #备份plist 以防之后操作失误
ʕ ◔ ϖ ◔ ʔ   ➜  Preferences cp ~/Library/Preferences/clowwindy.ShadowsocksX.plist ~/Desktop #取出这个plist文件到桌面
ʕ ◔ ϖ ◔ ʔ   ➜  Preferences cd ~/Desktop
ʕ ◔ ϖ ◔ ʔ   ➜  Desktop plutil -convert xml1 clowwindy.ShadowsocksX.plist -o ss.xml #plutil为mac自带解析plst的Shell工具

#查看自己的xml里面的<data>{YOUR SERVER INFO}</data>后 进行base64 decode解码

ʕ ◔ ϖ ◔ ʔ   ➜  Desktop echo {YOUR SERVER INFO}|base64 --decode #base64解码

#好了 你现在找到你的config.json了

```

哦,对了。如果你好奇我这个Shell主题是什么的话,可以戳[这里](https://github.com/scbizu/gopher)查看这个[ZSH](http://ohmyz.sh/)主题.
