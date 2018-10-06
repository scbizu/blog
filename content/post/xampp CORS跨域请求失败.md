---
categories:
- 伪运维相关
tags:
- Solution
- Xampp
title: xampp CORS跨域请求失败
---
## What is CORS



咳咳,CORS是一种跨域请求,全称是Cross Origin Resource Sharing;

如果 之前逛过我部落格的同学们可以发现,窝的文章分享侧栏就是因为CORS disable的原因 导致一些icon/font 显示不出来。

在F12->Console里面可以很清楚的看到这个error:



<blockquote>Font blocked from loading by Cross-Origin Resource Sharing policy: No 'Access-Control-Allow-Origin'</blockquote>







## How can i solve it



根据这个error 我们可以很清楚的看到是Request Header的锅

然而这个Req肯定是由服务器配置决定的

于是 自然而然地就想到了 <del>.htaccess</del>,啊呸,应该是httpd.conf文件



### Xampp



由于服务器配的是xampp(懒)   xampp的httpd.conf 位于/lampp/etc下

vim打开它

果然~~~

没有配置CORS;

(翻的好累hhhhhhh)

于是把CORS设置为*     lampp restart  问题就解决啦~



### My Configuration Code

```



    # ----------------------------------------------------------------------
    # CORS-enabled images (@crossorigin)
    # ----------------------------------------------------------------------
    # Send CORS headers if browsers request them; enabled by default for images.
    # developer.mozilla.org/en/CORS_Enabled_Image
    # blog.chromium.org/2011/07/using-cross-domain-images-in-webgl-and.html
    # hacks.mozilla.org/2011/11/using-cors-to-load-webgl-textures-from-cross-domain-images/
    # wiki.mozilla.org/Security/Reviews/crossoriginAttribute
    <IfModule mod_setenvif.c>
      <IfModule mod_headers.c>
        # mod_headers, y u no match by Content-Type?!
        <FilesMatch "\.(gif|png|jpe?g|svg|svgz|ico|webp)$">
          SetEnvIf Origin ":" IS_CORS
          Header set Access-Control-Allow-Origin "*" env=IS_CORS
        </FilesMatch>
      </IfModule>
    </IfModule>
    # ----------------------------------------------------------------------
    # Webfont access
    # ----------------------------------------------------------------------
    # Allow access from all domains for webfonts.
    # Alternatively you could only whitelist your
    # subdomains like "subdomain.example.com".
    <IfModule mod_headers.c>
      <FilesMatch "\.(ttf|ttc|otf|eot|woff|woff2|font.css|css|js)$">
        Header set Access-Control-Allow-Origin "*"
      </FilesMatch>
    </IfModule>

```



## end
