---
categories:
- 伪运维相关
tags:
- Solution
- Xampp
title: 改变mysql默认密码
---

###            [官方源POST戳这里](http://scnace.cc/dashboard/docs/reset-mysql-password.html)



用过的xampp的都应该知道,xampp的mysql/mariaDB的密码是设置为空的，在建站以及开发中这是不被推荐的。并且,安装大多数框架时,数据库密码是不可以为空的。所以,重置密码是很重要的。





## SETP1:





 确保你的服务器上的数据库服务是开着的。







## STEP2：





打开Linux里的终端(Terminal)







## STEP3:





使用 mysqladmin命令 `mysqladmin`来更改数据库里的root密码. 语法格式如下:





	mysqladmin --user=root password="yourpassword"





然后，你可以这么写:



 	sudo  /opt/lampp/bin/mysqladmin --user=root password="scnace"







## STEP4:



虽然是及时更改的,但是还是建议重启下服务:



	sudo /opt/lampp/lampp stop

    sudo /opt/lampp/lampp start









## END
