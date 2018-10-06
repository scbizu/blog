---
categories:
- 伪技术相关
- php

tags:
- Solution
- php
- jQuery
title: php&jQuery 实现动态链接
---

之前已经遇到过很多次这种问题了。

每次都要去看一遍sf上的解决方案,由于又不是一毛一样的问题,所以,每次都要自己再想一遍整个流程。

So，赶脚 很不爽，还是记录下来吧，下次调用起来简单点。。。



## 问题描述:



一直觉得不描述问题,直接Show me code 或是 大谈解决方案的博客就是耍流氓。

(我才不要耍流氓)正文开始:

在很多页面中用 **CELL** 或是 **LIST** 或是 **TABLE** 布局的时候,最常用的肯定是LOOP一个

<cell></cell>了,这样就可以实现动态布局了,可是当这个CELL还要附带一个动态链接时,问题来了,我们要怎么构建这个href呢~  没错,窝要说的就是这个问题。





## 解决方案(Without code)



这里先说下解决思路。。。。

首先,俺们当然要构造出这个Link啦,那么从服务器端取ID肯定是首选,每条CELL肯定会有其对应的动态ID记录,我们尝试把它拿过来。拿过来之后,我们要让你的每个CELL都有其自己的标志(构造附带这个ID的CELL),这边可以给这个DIV加for或者rel属性,或者anything U like。接下来给这个Div绑Click事件。当按下之后,我们要动态获取那个CELL ID,

怎么获取就是要通过之前你写下的属性了。然后直接通过Get传值——>跳转，OK。问题解决。

UPDATE:

当然还要记得,div不能设为ID,id选择器是不能绑多个事件的,要加改为class选择器。



## Code is here



<del>没时间解释了,直接上Code吧。</del>

(这边给div插入的是rel属性)

(Jquery+PHP 配置)

HTML:



      <div class="tinfo" rel="{$v['jobid']}">
         <div class="weui_cell_hd">
        <p> {$v['title']}</p>
         <div align="right" style="margin-top:-20px">
                <p style="color:red">悬赏:{$v['credit']}积分</p>
         </div>
         </div>


             <div class="weui_cell_bd weui_cell_primary">
              {if $v['pic1'] || $v['pic2'] || $v['pic3']}  
               <div style="width:300px;height:80px;margin-top:20px;margin-bottom:10px">  
                    {if $v['pic1']}  
                        <img src="{$v['pic1']}" alt="" style="width:100px;height:80px"/>
                    {/if}
                    {if $v['pic2']}
                        <img src="{$v['pic2']}" alt="" style="width:100px;height:80px;margin-left:110px;margin-top:-105px">
                    {/if}
                    {if $v['pic3']}
                        <img src="{$v['pic2']}" alt="" style="width:100px;height:80px;margin-left:230px;margin-top:-148px">
                    {/if}               
              </div>              
                {/if}
              <br/>    
              <p>{$v['content']} </p>
              <br/>           
                <p  style="color:#333">发布方相关:@{$v['name']} </p>
                </div>
      </div>



JQ:



      $(".tinfo").click(function(){
            var jobid=$(this).attr('rel')
            window.location.href="{php echo $this->createMobileUrl('ptjdetails');}"+"&jobid="+jobid;
        });









完毕~
