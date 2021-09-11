---
title: Prepared Statements
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



# MySQL的SQL预处理(Prepared)  

学习自[MySQL的SQL预处理(Prepared)](https://www.cnblogs.com/geaozhang/p/9891338.html#a)

> Prepared SQL Statement：SQL的执行、预编译处理语法、注意点

[toc]

## 一、SQL 语句的执行处理

### 1、即时 SQL

一条 SQL 在 DB 接收到最终执行完毕返回，大致的过程如下：

1. 词法和语义解析；

2. 优化 SQL 语句，制定执行计划；

3. 执行并返回结果；

​        如上，一条 SQL 直接是走流程处理，一次编译，单次运行，此类普通语句被称作 Immediate Statements （即时 SQL）。

### 2、预处理 SQL

​		但是，绝大多数情况下，某需求某一条 SQL 语句可能会被反复调用执行，或者每次执行的时候只有个别的值不同（比如 select 的 where 子句值不同，update 的 set 子句值不同，insert 的 values 值不同）。如果每次都需要经过上面的词法语义解析、语句优化、制定执行计划等，则效率就明显不行了。
​		所谓预编译语句就是将此类 SQL 语句中的值用占位符替代，可以视为将 SQL 语句模板化或者说参数化，一般称这类语句叫Prepared Statements。

​		预编译语句的优势在于归纳为：一次编译、多次运行，省去了解析优化等过程；此外预编译语句能防止 SQL 注入。

**注意：**

​		虽然可能是通过预处理 SQL 的方式一定程度的提高了效率，但是对于优化而言，最优的执行计划不是光靠 SQL 语句的模板化来实现的，往往还是需要通过具体值来预估出成本代价。

## 二、Prepared SQL Statement Syntax

​		MySQL 官方将 prepare、execute、deallocate 统称为 PREPARE STATEMENT。翻译也就习惯的称其为预处理语句。
　　MySQL 预处理语句的支持版本较早，所以我们目前普遍使用的 MySQL 版本都是支持这一语法的。
语法：

```
# 定义预处理语句
PREPARE stmt_name FROM preparable_stmt;
# 执行预处理语句
EXECUTE stmt_name [USING @var_name [, @var_name] ...];
# 删除(释放)定义
{DEALLOCATE | DROP} PREPARE stmt_name;
```

###  1、利用字符串定义预处理 SQL (直角三角形计算)

```
mysql> PREPARE stmt1 FROM 'SELECT SQRT(POW(?,2) + POW(?,2)) AS hypotenuse';
Query OK, 0 rows affected (0.00 sec)
Statement prepared

mysql> SET @a = 3;
Query OK, 0 rows affected (0.00 sec)

mysql> SET @b = 4;                                                   
Query OK, 0 rows affected (0.00 sec)

mysql> EXECUTE stmt1 USING @a, @b;
+------------+
| hypotenuse |
+------------+
|          5 |
+------------+
1 row in set (0.00 sec)

mysql> DEALLOCATE PREPARE stmt1;                                     
Query OK, 0 rows affected (0.00 sec)
```

###  2、利用变量定义预处理 SQL (直角三角形计算)

```
mysql> SET @s = 'SELECT SQRT(POW(?,2) + POW(?,2)) AS hypotenuse';
Query OK, 0 rows affected (0.00 sec)

mysql> PREPARE stmt2 FROM @s;
Query OK, 0 rows affected (0.00 sec)
Statement prepared

mysql> SET @c = 6;
Query OK, 0 rows affected (0.00 sec)

mysql> SET @d = 8;
Query OK, 0 rows affected (0.00 sec)

mysql> EXECUTE stmt2 USING @c, @d;
+------------+
| hypotenuse |
+------------+
|         10 |
+------------+
1 row in set (0.00 sec)

mysql> DEALLOCATE PREPARE stmt2;
Query OK, 0 rows affected (0.00 sec)
```

###  3、解决无法传参问题

　　我们知道，对于 LIMIT 子句中的值，必须是常量，不得使用变量，也就是说不能使用：SELECT * FROM TABLE LIMIT @skip, @numrows; 如此，就可以是用 PREPARE 语句解决此问题。

```
mysql> SET @skip = 100; SET @numrows = 3;
Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

mysql> SELECT * FROM t1 LIMIT @skip, @numrows;
ERROR 1064 (42000): You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near '@skip, @numrows' at line 1

mysql> PREPARE stmt3 FROM "SELECT * FROM t1 LIMIT ?, ?";
Query OK, 0 rows affected (0.00 sec)
Statement prepared

mysql> EXECUTE stmt3 USING @skip, @numrows;
+-----+--------+
| a   | filler |
+-----+--------+
| 100 | filler |
| 101 | filler |
| 102 | filler |
+-----+--------+
3 rows in set (0.00 sec)

mysql> DEALLOCATE PREPARE stmt3;
Query OK, 0 rows affected (0.00 sec)
```

 　   如此一来，结合2中介绍的利用变量定义预处理 SQL 也就基本解决了传参时语法报错问题了，类似的：用变量传参做表名时，MySQL 会把变量名当做表名，这样既不是本意，也会是语法错误，在 SQL Server 的解决办法是利用字符串拼接穿插变量进行传参，再将整条 SQL 语句作为变量，最后是用 sp_executesql 调用该拼接 SQL 执行，而 Prepared SQL Statement 可谓异曲同工之妙。

```
mysql> SET @table = 't2';
Query OK, 0 rows affected (0.00 sec)

mysql> SET @s = CONCAT('SELECT * FROM ', @table);
Query OK, 0 rows affected (0.00 sec)

mysql> PREPARE stmt4 FROM @s;
Query OK, 0 rows affected (0.00 sec)
Statement prepared

mysql> EXECUTE stmt4;
+------+-------+-------+
| id   | score | grade |
+------+-------+-------+
|    1 |    99 | A     |
|    2 |    81 | B     |
|    3 |    55 | D     |
|    4 |    69 | C     |
+------+-------+-------+
4 rows in set (0.00 sec)

mysql> DROP PREPARE stmt4;
Query OK, 0 rows affected (0.00 sec)
```

## 三、预处理 SQL 使用注意点

1、stmt_name 作为 preparable_stmt 的接收者，唯一标识，不区分大小写。
2、preparable_stmt 语句中的 ? 是个占位符，所代表的是一个字符串，不需要将 ? 用引号包含起来。
3、定义一个已存在的 stmt_name ，原有的将被立即释放，类似于变量的重新赋值。
4、PREPARE stmt_name 的作用域是session级

![img](https://gitee.com/h1ler/tuci/raw/master/null/1113510-20181101192302151-157684852.png)
　　可以通过 max_prepared_stmt_count 变量来控制全局最大的存储的预处理语句。

```
mysql> show variables like 'max_prepared%';
+-------------------------+-------+
| Variable_name           | Value |
+-------------------------+-------+
| max_prepared_stmt_count | 16382 |
+-------------------------+-------+
1 row in set (0.00 sec)
```

 　   预处理编译 SQL 是占用资源的，所以在使用后注意及时使用 DEALLOCATE PREPARE 释放资源，这是一个好习惯。