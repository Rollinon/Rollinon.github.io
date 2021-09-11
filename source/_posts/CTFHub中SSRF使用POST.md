---
title: CTFHub中SSRF使用POST
date: {{ date }}
top: false
cover: false
password:
toc: true
mathjax: true
summary:
tags: [SSRF,CTF]
categories: [SSRF,CTF]
---

## 审计flag.php

这里有一个点是`$_SERVER["REMOTE_ADDR"]`是“可靠”的请求端ip地址，是服务端在进行TCP握手时获得的，从技术上来说是几乎不可能伪造的，与`$_SERVER["HTTP_X_FORWARDED_FOR"]`和`$_SERVER["HTTP_CLIENT_IP"]`这两个可以通过修改http报头轻松伪造的参数不同。



**记得gopher的默认端口是70，如果要发送请求到其它端口，记得修改**

简化一下流程：

```
1.侦测注入点
2.找到攻击对象，如“flag.php”
3.构造gopher数据（最常见的就是通过本地请求，本地抓包，然后手动或者脚本构造）
4.进行攻击
```

> 参考
>
> https://liloong3t.com/2021/02/16/2021-2-16-ssrf-shi-yong-gopher-xie-yi/

