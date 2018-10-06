---
categories:
- 伪技术相关
- php
tags:
- Solution
- php
title: php emoji表情无法获取问题
---

## why ?



带Emoji表情的用户名简直就是社交用户三大毒瘤之一(e.g. Instagram,facebook,and wechat);



然而,跟用户说 我不让你用Emoji表情 可能会极大程度激发用户的逆反心理(一甩手,劳资不用你的东西了!);



所以,该解决的东西还是要解决;



特别是对于那些社交APP衍生物 ;(我才没有在说微信公众号 和 某人调Instagram API的辣鸡东西呢;





## How TO?



因为是在Wechat上遇到的问题,我就去知乎 和 V2 找了下解决方案(什么你说还有天朝SO--SF?);

嗯 没错 因为V2er们的指引  找到了这篇SO 的Post ;

(为什么我会以为我的SEO会在SO之前 是不是太naive了2333)

但还是要引用过来,万一哪天SO 挂(GFW)了呢~

所以,

[这里是原po;](http://stackoverflow.com/questions/12807176/php-writing-a-simple-removeemoji-function)





## Code Here



//原理:使用正则表达去匹配到所有的Emoji表情符号   好拼~

```

    public static function removeEmoji($text) {

        $clean_text = "";

        // Match Emoticons
        $regexEmoticons = '/[\x{1F600}-\x{1F64F}]/u';
        $clean_text = preg_replace($regexEmoticons, '', $text);

        // Match Miscellaneous Symbols and Pictographs
        $regexSymbols = '/[\x{1F300}-\x{1F5FF}]/u';
        $clean_text = preg_replace($regexSymbols, '', $clean_text);

        // Match Transport And Map Symbols
        $regexTransport = '/[\x{1F680}-\x{1F6FF}]/u';
        $clean_text = preg_replace($regexTransport, '', $clean_text);

        // Match Miscellaneous Symbols
        $regexMisc = '/[\x{2600}-\x{26FF}]/u';
        $clean_text = preg_replace($regexMisc, '', $clean_text);

        // Match Dingbats
        $regexDingbats = '/[\x{2700}-\x{27BF}]/u';
        $clean_text = preg_replace($regexDingbats, '', $clean_text);

        return $clean_text;
    }


```




## The End
