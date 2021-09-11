---
title: SSRF应用场景
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



## SSRF在Python中的利用

- urllib曾爆出CVE-2019-9740、CVE-2019-9947两个漏洞，这两个漏洞都是urllib(urllib2)的CRLF漏洞，只是触发点不一样，其影响范围都在urllib2 in Python 2.x through 2.7.16 and urllib in Python 3.x through  3.7.3之间，目前大部分服务器的python2版本都在2.7.10以下，python3都在3.6.x，这两个CRLF漏洞的影响力就非常可观了。其实之前还有一个CVE-2016-5699，同样的urllib（urllib2）的CRLF问题，但是由于时间比较早，影响范围没有这两个大
- 除开CRLF之外，urllib还有一个鲜有人知的漏洞CVE-2019-9948，该漏洞只影响urllib，范围在Python  2.x到2.7.16，这个版本间的urllib支持local_file/local-file协议，可以读取任意文件，如果file协议被禁止后，不妨试试这个协议来读取文件。

## SSRF在JAVA中的利用

## 30x跳转

## URL解析绕过

## DNS rebinding

> 参考
>
> https://security.tencent.com/index.php/blog/msg/179

https://websec.readthedocs.io/zh/latest/vuln/ssrf.html

https://websec.readthedocs.io/zh/latest/vuln/index.html

https://www.evi1s.com/archives/140/

https://www.anquanke.com/post/id/226240

http://www.feidao.site/wordpress/?p=2116

