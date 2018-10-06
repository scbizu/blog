---
categories:
- 伪技术相关
- golang
tags:
- repo
- golang
title: golang的一个OCR库
---
嗯，这篇要记录的是 Golang的一个OCR库。名字是 [gosseract](https://github.com/otiai10/gosseract) (基于用途最广泛的一个OCR库[tesseract](https://github.com/tesseract-ocr/tesseract))

遇到的问题是 按照 说明





  1. install tesseract-ocr


  2. install go


  3. install gosseract


    * `go get github.com/otiai10/gosseract`





  4. install mint for testing


    * `go get github.com/otiai10/mint`





  5. run the tests at first



在当前目录 go test ./...          之后。。。

出现了build error:



    tesseract/tess.cpp:1:31: fatal error: tesseract/baseapi.h: No such file or directory
    compilation terminated.
    FAIL    github.com/otiai10/gosseract [build failed]



或者:



    tesseract/tess.cpp:2:34: fatal error: leptonica/allheaders.h: No such file or directory
    compilation terminated.
    FAIL    github.com/otiai10/gosseract [build failed]





原因:

需要 `libleptonica-dev ``     libtesseract-dev `  依赖。

在Linux(**Debian系**)下,可以通过:



    apt-get install libleptonica-dev 

    apt-get install  libtesseract-dev



来安装依赖。



坑:有可能经过repo 的 readme的误导,很多同学都会选择compile tesseract from source 这真的是个深坑  这么做简直是吃力不讨好的行为 走了远路。(除非包管理工具真的没有这两货



参考: [#issue45](https://github.com/otiai10/gosseract/issues/45)
