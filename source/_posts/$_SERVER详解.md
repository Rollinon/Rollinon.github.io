---
title: $_SERVER详解
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

# $_SERVER

[官方文档](https://www.php.net/manual/zh/reserved.variables.server.php)

$_SERVER -- $HTTP_SERVER_VARS [已删除] — 服务器和执行环境信息

------

## 目录

如果以[命令行](https://www.php.net/manual/zh/features.commandline.php)方式运行 PHP，下面列出的元素几乎没有有效的(或是没有任何实际意义的)

$\_SERVER['HTTP_ACCEPT_LANGUAGE']

浏览器语言 
$\_SERVER['REMOTE_ADDR'] 

当前用户 IP 。 
$\_SERVER['REMOTE_HOST'] 

当前用户主机名 
$\_SERVER['REQUEST_URI'] 

URL

$\_SERVER['REMOTE_PORT'] 

端口。 
$\_SERVER['SERVER_NAME'] 

服务器主机的名称。 
$\_SERVER['PHP_SELF']

正在执行脚本的文件名 
$\_SERVER['argv']

传递给该脚本的参数。 
$\_SERVER['argc'] 

传递给程序的命令行参数的个数。 
$\_SERVER['GATEWAY_INTERFACE']

CGI 规范的版本。 
$\_SERVER['SERVER_SOFTWARE'] 

服务器标识的字串 
$\_SERVER['SERVER_PROTOCOL'] 

请求页面时通信协议的名称和版本 
$\_SERVER['REQUEST_METHOD']

访问页面时的请求方法 
$\_SERVER['QUERY_STRING'] 

查询(query)的字符串。 
$\_SERVER['DOCUMENT_ROOT'] 

当前运行脚本所在的文档根目录 
$\_SERVER['HTTP_ACCEPT'] 

当前请求的 Accept: 头部的内容。 
$\_SERVER['HTTP_ACCEPT_CHARSET'] 

当前请求的 Accept-Charset: 头部的内容。 
$\_SERVER['HTTP_ACCEPT_ENCODING'] 

当前请求的 Accept-Encoding: 头部的内容 
$\_SERVER['HTTP_CONNECTION'] 

当前请求的 Connection: 头部的内容。例如：“Keep-Alive”。 
$\_SERVER['HTTP_HOST'] 

当前请求的 Host: 头部的内容。 
$\_SERVER['HTTP_REFERER'] 

链接到当前页面的前一页面的 URL 地址。 
$\_SERVER['HTTP_USER_AGENT'] 

当前请求的 User_Agent: 头部的内容。 
$\_SERVER['HTTPS']

如果通过https访问,则被设为一个非空的值(on)，否则返回off 
$\_SERVER['SCRIPT_FILENAME'] 

当前执行脚本的绝对路径名。 
$\_SERVER['SERVER_ADMIN'] 

管理员信息 
$\_SERVER['SERVER_PORT'] 

服务器所使用的端口 
$\_SERVER['SERVER_SIGNATURE'] 

包含服务器版本和虚拟主机名的字符串。 
$\_SERVER['PATH_TRANSLATED'] 

当前脚本所在文件系统（不是文档根目录）的基本路径。 
$\_SERVER['SCRIPT_NAME'] 

包含当前脚本的路径。这在页面需要指向自己时非常有用。 
$\_SERVER['PHP_AUTH_USER'] 

当 PHP 运行在 Apache 模块方式下，并且正在使用 HTTP 认证功能，这个变量便是用户输入的用户名。 
$\_SERVER['PHP_AUTH_PW'] 

当 PHP 运行在 Apache 模块方式下，并且正在使用 HTTP 认证功能，这个变量便是用户输入的密码。 
$\_SERVER['AUTH_TYPE'] 

当 PHP 运行在 Apache 模块方式下，并且正在使用 HTTP 认证功能，这个变量便是认证的类型