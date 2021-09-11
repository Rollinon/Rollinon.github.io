---
title: Git泄露
date: {{ date }}
top: false
cover: false
password:
toc: true
mathjax: true
summary:
tags: Git
categories: Git
---



# CTFHub-Git泄露

一般思路：用dirsearch扫描发现git泄露

![image-20201023023245829](https://gitee.com/h1ler/tuci/raw/master/null/image-20201023023245829.png)

![image-20201023023226766](https://gitee.com/h1ler/tuci/raw/master/null/image-20201023023226766.png)

## Log

- 知识点：详见[Git 基础 - 查看提交历史](https://git-scm.com/book/zh/v2/Git-%E5%9F%BA%E7%A1%80-%E6%9F%A5%E7%9C%8B%E6%8F%90%E4%BA%A4%E5%8E%86%E5%8F%B2)、[git diff](https://git-scm.com/docs/git-diff)

- 使用GitHack工具处理Git泄露情况![image-20201023023626635](https://gitee.com/h1ler/tuci/raw/master/null/image-20201023023626635.png)

- 切换到爬下来的目录中用`git log`查看日志![image-20201023024006671](https://gitee.com/h1ler/tuci/raw/master/null/image-20201023024006671.png)

- 得到Flag

  - 方法一：发现提交说明中有add flag，利用`git diff`来查看commits之间的不同![image-20201023024655911](https://gitee.com/h1ler/tuci/raw/master/null/image-20201023024655911.png)

  - 方法二：用`git reset --hard`退回提交说明为add flag的版本![image-20201023024951095](https://gitee.com/h1ler/tuci/raw/master/null/image-20201023024951095.png)

    即可将add flag版本克隆在本地文件夹中![image-20201023025225404](https://gitee.com/h1ler/tuci/raw/master/null/image-20201023025225404.png)

## Stash

- 知识点：[git-stash用法小结](https://www.cnblogs.com/tocy/p/git-stash-reference.html)
- 操作同上，进入文件夹后使用`git log`![image-20201023025747482](https://gitee.com/h1ler/tuci/raw/master/null/image-20201023025747482.png)
- `git diff`并没有flag信息![image-20201023025905370](https://gitee.com/h1ler/tuci/raw/master/null/image-20201023025905370.png)
- `git stash list`操作，查看现有stash![image-20201023030022983](https://gitee.com/h1ler/tuci/raw/master/null/image-20201023030022983.png)
- `git stash apply`重新应用缓存的stash![image-20201023030151666](https://gitee.com/h1ler/tuci/raw/master/null/image-20201023030151666.png)
- `git diff`重新比对得到flag![image-20201023030323906](https://gitee.com/h1ler/tuci/raw/master/null/image-20201023030323906.png)

## Index

