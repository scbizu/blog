---
categories:
- 伪运维相关
tags:
- Solution
- Xampp
title: 开启Xampp phpmyadmin外网访问
---

Xampp确实是样好东西,既然号称是Apache Friend,那么当然也帮我们搭好了PHP的集成环境。



然而, 在Xampp的更新中,phpmyadmin的外网访问被403了，内网还是可以访问的。考虑到了 更安全 更专业的建站,当然,这种做法是被提倡和被建议的.



但是,有些时候是真的要上外网 phpmyadmin 啊。



所以,解决方案如下所示(用于应急):

先到如下目录`/opt/lampp/etc/extra`

然后找到如下文件`httpd-xampp.conf`

Edit  it！

一直page down到最后,你会发现**LocationMatch**标签里面 有个通配表达式,对PHPMYADMIN就在里面,还在一起的还有Security和server-stats等，这些都被403了。

你要做的就是 在**LocationMatch**里面把**require local**改为**require all granted**

最后！

不要忘了！

Restart xampp！
