---
title: SSRF_Bypass
date: {{ date }}
top: false
cover: false
password:
toc: true
mathjax: true
summary:
tags: SSRF
categories: SSRF
---



# SSRF_Bypass

## URL

两个函数一个是url_parser根据url解析出它的host,query,path等信息,第二个就是curl的扩展我们知道curl在php里面可以去用来捕获一个页面取回并输出,那么针对url_parser和curl会有一个识别host的误差,我们可以结合这两个识别host的不同.对地址过滤的情况进行一个bypass.

![img](https://gitee.com/h1ler/blogimage/raw/master/img/104969439.png)

参考[知识盒子](https://zhishihezi.net/box/b826e3c021c5dc367665f743ae5fa14b)SSRF题目之地址绕过

## 数字IP

尝试将ip地址转换为进制的方式进行绕过,127.0.0.1转换为16进制是0x7F000001

## DNS重绑定

https://zhuanlan.zhihu.com/p/89426041