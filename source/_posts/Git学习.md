---
title: Git学习
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



# Git

- Leaning from [Git book](https://git-scm.com/book/zh/v2)



## 2. Git 基础



### 2.3 Git 基础 - 查看提交历史

- `git log` 命令

  ![image-20201023013143508](https://gitee.com/h1ler/tuci/raw/master/null/image-20201023013143508.png)

  - 该命令列出每个提交的 SHA-1 校验值、作者的名字和电子邮件地址、提交时间以及提交说明。

  - 不传入任何参数的默认情况下，`git log` 会按时间先后顺序列出所有的提交，最近的更新排在最上面。

  - 选项 `-p` 或 `--patch` 。它会显示每次提交所引入的差异（按 **补丁** 的格式输出），你也可以限制显示的日志条目数量，例如使用 `-2` 选项来只显示最近的两次提交。![image-20201023013249918](https://gitee.com/h1ler/tuci/raw/master/null/image-20201023013249918.png)

  - 选项 `--stat` 。当进行代码审查，或者快速浏览某个搭档的提交所带来的变化的时候，这个参数就非常有用了。你也可以为 `git log` 附带一系列的总结性选项。 比如你想看到每次提交的简略统计信息：![image-20201023013424138](https://gitee.com/h1ler/tuci/raw/master/null/image-20201023013424138.png)正如你所看到的，`--stat` 选项在每次提交的下面列出所有被修改过的文件、有多少文件被修改了以及被修改过的文件的哪些行被移除或是添加了。 在每次提交的最后还有一个总结。

  - 选项 `--pretty`。这个选项可以使用**不同于默认格式**的方式展示提交历史。 这个选项有一些内建的子选项供你使用。

    -  比如 `oneline` 会将每个提交放在一行显示，在浏览大量的提交时非常有用。 另外还有 `short`，`full` 和 `fuller` 子选项，与`oneline`展示信息的格式基本一致，详尽程度不一。![image-20201023013831955](https://gitee.com/h1ler/tuci/raw/master/null/image-20201023013831955.png)

    - 子选项 `format` ，可以定制记录的显示格式![image-20201023014446505](https://gitee.com/h1ler/tuci/raw/master/null/image-20201023014446505.png)`format` 接受的常用格式占位符的写法及其代表的意义可参考官方文档[常用的选项](https://git-scm.com/book/zh/v2/ch00/pretty_format)

  -  `--since` 和 `--until` 这种按照时间作限制的选项很有用。 例如，下面的命令会列出最近两周的所有提交：

    ```shell
    $ git log --since=2.weeks
    ```

    该命令可用的格式十分丰富——可以是类似 `"2008-01-15"` 的具体的某一天，也可以是类似 `"2 years 1 day 3 minutes ago"` 的相对日期。

  -  `--author` 选项显示指定作者的提交.

  -  `--grep` 选项搜索提交说明中的关键字。

  - 另一个非常有用的过滤器是 `-S`（俗称“pickaxe”选项，取“用鹤嘴锄在土里捡石头”之意）， 它接受一个字符串参数，并且只会显示那些添加或删除了该字符串的提交。 假设你想找出添加或删除了对某一个特定函数的引用的提交，可以调用：

    ```shell
    $ git log -S function_name
    ```



## [git-stash用法小结](https://www.cnblogs.com/tocy/p/git-stash-reference.html)



