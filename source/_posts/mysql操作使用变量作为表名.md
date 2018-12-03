---
title: mysql操作使用变量作为表名
date: 2018-06-08 10:42:11
categories:
- MySQL
tags:
---

<!-- more -->

> 版权声明：本文为博主原创文章，未经博主允许不得转载。

使用python操作mysql数据库时想实现如下效果：
将sql语句中的表名用一个变量代替，实现批量操作表

    sql = "select * from sets where number = '%s' "

将sql语句中的sets和number用变量代替，实现自动操作多个表：

```python
table = "sets"
attr = "number"
sql = "select * from "+table+" where "+attr+" = '%s'"
# 成功运行，达到要求
```

> 其实就是采用字符串操作中的语句拼接技术