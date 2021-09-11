---
title: php伪协议总结
date: {{ date }}
top: false
cover: false
password:
toc: true
mathjax: true
summary:
tags: PHP
categories: PHP
---

# PHP伪协议

php配置

allow_url_fopen=On

allow_url_include=On

## php://input

可直接获取POST传输的数据

不用键与值的形式，只写代码即可

可以用作命令执行，写一句话木马

## php://filter

常见用法

```php
php://filter/read=convert.base64-encode/resource=文件路径
```

- 非php语法文件include失败，直接输出源码内容
- php语法文件include成功，直接执行

## file://

file:// 与php:filter类似，访问本地文件，但是只能传入绝对路径

## phar://

查找指定压缩包内的文件

## zip://

与phar://类似

注意

1. 只能传入绝对路径
2. 要用#分隔压缩包和压缩包里的内容，#用url编码%23

## data

与input类似

用法

1. 直接写入代码

   ```php
   data:text/plain,<?php phpinfo();?>
   ```

2. 使用base64编码

   ```php
   data:text/plain;base64,编码后的php代码
   ```

   注意：base64编码后的加号和等号要手动进行url编码，否则无法识别

## 总结

![img](https://gitee.com/h1ler/blogimage/raw/master/img/20180606180044611)

