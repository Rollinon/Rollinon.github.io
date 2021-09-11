---
title: RCE防护
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



代码注入的防治：

1. 升级到PHP7.1，该版本对大部分常见的执行动态代码的函数进行了封堵。
2. php.ini中，关闭`allow_url_fopen`。在打开它的情况下，可以通过`phar://`等协议丢给`include`，让其执行动态代码。
3. php.ini中，通过`disable_functions`关闭`exec`,`passthru`,`shell_exec`,`system`等函数，禁止PHP调用外部程序。
4. 永远不要在代码中使用`eval`。
5. 设置好上传文件夹的权限，禁止从该文件夹执行代码。
6. include文件的时候，注意文件的来源；需要动态include时做好参数过滤。
7. 对于`preg_replace()`函数，要放弃使用/e修饰符，也可以使用`preg_replace_callback()`函数代替。如果一定要使用该函数，请保证第二个参数中，对于正则匹配出的对象用单引号包裹，因为`mixed preg_replace ( mixed pattern, mixed replacement, mixed subject [, int limit]) /e`修正符使preg_replace()将replacement参数当作PHP代码（在适当的逆向引用替换完之后）。

