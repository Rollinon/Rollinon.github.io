---
title: HTTP协议
date: {{ date }}
top: false
cover: false
password:
toc: true
mathjax: true
summary:
tags: 计算机网络
categories: 计算机网络
---



# HTTP协议

## **重要性**

- *无论是以后用webservice，还是用rest做大型架构，都离不开对HTTP协议的认识。*

- *甚至可以简化地说：*
  *ebService = HTTP协议 + XML*
  *Rest = HTTP协议 + json*
  *各种API也一般是用HTTP + XML/json来实现的*

- *在对HTTP协议有了了解之后，学习Ajax是非常容易理解的*

## **什么是协议**

- *计算机中的协议和现实中的协议是一样的，一式双份/多份。双方/多方都遵从共同的一个规范，这个规范就可以称为协议。*

- <u>ftp、http、stmp、pop、tcp/ip协议</u>

## **什么是http协议**

- *http协议即按照一定规则，向服务器请求数据或发送数据，而服务器按一定规则返回数据。*

##   **HTTP工作流程**

- ![wps1](https://gitee.com/h1ler/tuci/raw/master/null/wps1.png)

## **HTTP请求信息和响应信息的格式**

![](https://gitee.com/h1ler/tuci/raw/master/null/image-20200728160038988.png)

![image-20200728160443257](https://gitee.com/h1ler/tuci/raw/master/null/image-20200728160443257.png)

- 请求：
  
  
  
  - 请求行
    - 请求方法
      - GET
      
      - POST
      
        与GET基本一致，只是不返回内容。
      
        例如我们只是确认一个内容（例如照片）是否正常存在，不需要返回照片内容，则用HEAD
      
      - HEAD
      
      - PUT
      
      - DELETE
      
      - TRACE
      
        当使用代理上网时，用代理访问new.163.com，用TRACE可以测试代理是否修改你的HTTP请求，163.com的服务器会将最后收到的请求返回
      
      - OPTIONS
      
        返回服务器可用的请求方法
      
      >  注意：这些请求方法虽然HTTP协议里规定的，但WebService未必允许或支持这些方法
      
    - 请求路径
    
    - 所用的协议
    
      目前一般是HTTP/1.1，而0.9和10版本已经基本不用。
    
  - 请求头信息
  
    - 格式  Key:Value
  
    - Host
  
    - User-agent:    用户代理  
  
      向访问网站提供你所使用的浏览器类型及版本、操作系统及版本、浏览器内核、等信息的标识（爬虫可利用此伪装成用户）
  
    - Referer    告诉服务器该网页是从哪个页面链接过来的
  
    - X-Forwarded-For    用来识别通过[HTTP](https://baike.baidu.com/item/HTTP)[代理](https://baike.baidu.com/item/代理)或[负载均衡](https://baike.baidu.com/item/负载均衡)方式连接到[Web服务器](https://baike.baidu.com/item/Web服务器)的客户端最原始的[IP地址](https://baike.baidu.com/item/IP地址)的HTTP请求头字段(可利用伪造任意的IP地址)
  
    - Content-Type
  
    - Content-Length    接下来主体的长度
  
  - 请求主体信息（可选）

> 问：浏览器能发送HTTP协议，HTTP协议一定要浏览器来发送吗？
>
> 答：不是，HTTP既然是一种协议，那么只要满足这种协议，什么工具都可以发。

- 响应：

  - 响应行

    - 协议
    - 状态码
    - 状态文字

  - 响应头信息

    - 格式  Key:Value

    - Content-Type
    - Content-Length    接下来主体的长度

  - 响应主体信息(也可能没有)

> 作业：用telnet来完成HTTP协议的POST请求
>
> ![image-20200728164614323](https://gitee.com/h1ler/tuci/raw/master/null/image-20200728164614323.png)



![image-20200729004054696](https://gitee.com/h1ler/tuci/raw/master/null/hLj2cPlvxSo79EH.png)

## **状态码，状态文字**

状态码是用来反应服务器响应情况的。

最常见的如 200 OK，404 NOT FOUND

![image-20200729011518611](https://gitee.com/h1ler/tuci/raw/master/null/image-20200729011518611.png)

- 常见状态码

  200	-	服务器成功返回页面

  301/302	-	永久/临时重定向

  304 NOT Modified	-	未修改

  **307	-	重定向中保持原有的请求数据**

  失败的状态码：

  404	-	请求的网页不存在

  503	-	服务器暂时不可用
  
  500	-	服务器内部错误