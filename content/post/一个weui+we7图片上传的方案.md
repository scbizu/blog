---
title: 一个weui+we7图片上传的方案
---

### 写在前面：



weui(微信UI)地址[传送门](https://github.com/weui/weui/wiki)

we7  6.0 现已停止维护 为防止侵权 这边就不给传送门辣(百度云盘搜就可以找到,这就不关我的事了)





### 此文章要解决的问题：



RT, 相信用过we7的开发者都知道吧，作为一个php wechat的开源开发框架,确实是易用易入门的。但是，卧槽，[你们的wiki](http://www.we7.cc/manual/)是什么鬼啊,这太懒了啊  demo 和e.g.写得简单也就算了  但是你参数说明写清楚啊(虽然知道你们wiki是抓的代码注释,但我不管！）。。。（以上吐槽完毕,进入正文。）

这篇文章涉及到的是App端的图片(file,所以也可以是media)上传。我们先来看看we7手册给的相关内容(以下摘自we7.cc/manual):



<blockquote>先是一个表单生成模板里面的上传图片控(组)件(Component):</blockquote>



![1](http://139.129.37.170/wordpress/wp-content/uploads/2016/02/1.png)

啥 现在都HTML5了 你跟我说不能preview？果断 不能忍啊！所以一气之下 弃之！

好了 踩坑开始~~~







### 那么我该怎么做呢？



虽然我们放弃了用组件 但是我们还可以参考一下其Code啊(取之精华,弃之糟粕2333

根据we7手册说的架构 开始找啊找。。。。。

![2](http://139.129.37.170/wordpress/wp-content/uploads/2016/02/2.png)

然而 有个卵啊 根本找不到需要的image控件啊！

(后悔一开始没用IDE的global search 明明被这该死的manual坑了这么多次了

开始global search这个函数  幸运的是 在./web/common/里面发现了它！哦  意思是基础控件 欺负我傻咯。。 找到这个函数,看了一眼代码(因为不用看$opinion和$default),所以代码的阅读可以很快。我们发现了他的机制：就是uploadFile啊,没什么特别的, 只是要注意一下tomedia()这个函数,它可以强转http绝对路径,之后会用到的。

接下来 开始自己写form(注意:为确保文件的正确上传,请确保设置form表单属性  enctype="multipart/form-data")  根据weui的写法 用一下他的样式表 weui的安装与应用这边不说了 wiki里有写用nodejs用npm下下来用就好了。如下：



    <code>
      <div class="weui_cell_bd weui_cell_primary">
     <div class="weui_uploader">
     <div class="weui_uploader_hd weui_cell" >
     <!--div class="weui_cell_bd weui_cell_primary">学生证信息上传</div-->
     <!--div class="weui_cell_ft">3/5</div-->
     </div>
     <div class="weui_uploader_bd">
     <ul class="weui_uploader_files" id="fpic_p">
     </ul>
     <div class="weui_uploader_input_wrp">
     <input class="weui_uploader_input" type="file" accept="image/jpg,image/jpeg,image/png,image/gif" name="fpic" id="t_fpic">
     </div>
     </div>
     </div>
     </div>

    </code>





这边是单图上传的实例  多图加个mutiple就可以,这边设id当然是为了用jquery 实现preview啊。

Jquery处理图片的关键函数如下:



    <code>
    //Preview The pic scard
     function readPic_s(input){
     if(input.files && input.files[0]){
     var reader=new FileReader();

     reader.onload=function(e){
     $("#tscard_p").append("<li class='weui_uploader_file' style='background-image:url("+e.target.result+")'><li>");
     }
     reader.readAsDataURL(input.files[0]);
     }
    }

    </code>



这边用了h5的**FileReader()** 具体可以参考[这里](https://developer.mozilla.org/zh-CN/docs/Web/API/FileReader)，再配合下weui的<ul><li><li><ul>就可以实现“不用上传”就可以预览图片了。**注意一下这里background-image的使用**。

以上 第一步装逼完成，也是不同于内置控件的一步。



下一步  开始与服务器交互(由于是we7的框架 所以显然是以PHP为例 )

用脚趾头想想都知道input标签里面要加name来标记这次上传，这边value就不用了 想知道为什么可以戳[这里](http://www.w3school.com.cn/jsref/dom_obj_fileupload.asp)。 我们在site对应的这个控制器里面开始接收到这个FILE。 我们通过var_dump($filename)看出,我们已经成功接到这个file(image)的二进制数据了(和tmp文件也会暂时留下)  然后，接下来又迷茫了。我们的image的最终归宿肯定是attachment目录啊，这样就少一个upload函数  难道要自己写？好在这次手册派上用场了，我们查下file下的file_upload()函数:

![3](http://139.129.37.170/wordpress/wp-content/uploads/2016/02/3.png)

但是 这次学乖了 我去看了看这个函数的source code 。找到了如下的信息：



    <code>
    function file_upload($file, $type = 'image', $name = '') {
     if(empty($file)) {
     return error(-1, '没有上传内容');
     }
     if(!in_array($type, array('image','audio'))) {
     return error(-1, '未知的上传类型');
     }

     global $_W;

     if (empty($_W['uploadsetting'][$type])) {
     $_W['uploadsetting'] = array();
     $_W['uploadsetting'][$type]['folder'] = "{$type}s/{$_W['uniacid']}";
     $_W['uploadsetting'][$type]['extentions'] = $_W['config']['upload'][$type]['extentions'];
     $_W['uploadsetting'][$type]['limit'] = $_W['config']['upload'][$type]['limit'];
     }
     $settings = $_W['uploadsetting'];

     $extention = pathinfo($file['name'], PATHINFO_EXTENSION);
     if(!in_array(strtolower($extention), $settings[$type]['extentions'])) {
     return error(-1, '不允许上传此类文件');
     }
     if(!empty($settings[$type]['limit']) && $settings[$type]['limit'] * 1024 < filesize($file['tmp_name'])) {
     return error(-1, "上传的文件超过大小限制，请上传小于 {$settings[$type]['limit']}k 的文件");
     }
     $result = array();

     if(empty($name) || $name == 'auto') {
     $result['path'] = "{$settings[$type]['folder']}/" . date('Y/m/');
     mkdirs(ATTACHMENT_ROOT . '/' . $result['path']);
     do {
     $filename = random(30) . ".{$extention}";
     } while(file_exists(ATTACHMENT_ROOT . '/' . $result['path'] . $filename));
     $result['path'] .= $filename;
     } else {
     $result['path'] = $name . '.' .$extention;
     }

     if(!file_move($file['tmp_name'], ATTACHMENT_ROOT . '/' . $result['path'])) {
     return error(-1, '保存上传文件失败');
     }

     $result['success'] = true;
     return $result;
    }

    </code>



根据source code 提供的参数表,我们开始搓我们需要的参数，(这边report手册的一个笔误吧应该 $_FILE应该改为$_FILES),然后我们成功搓出了我们要的参数。。。



    <code>
    var_dump(file_upload($_FILES['scard'],'image',$_GPC['scard']));

    </code>



虽然 sourcecode里面已经结构写得很清楚了，但是这边我们继续dump一下，这样就可以很清楚地看到path和success的key了。

找到path了一切就简单了，我们先tomedia()一下。这样就 得到了绝对地址的path  这样就可以存数据库了，下次就可以很方便的读取到了。



本篇是对we7 wiki的补充和解说  如有不对请拍砖！

还有，这篇真的是我原创啊  转的话请注明下出处(虽然我觉得并没什么好转的。)。
