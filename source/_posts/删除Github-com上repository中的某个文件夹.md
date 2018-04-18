---
title: 删除Github.com上repository中的某个文件夹
date: 2018-04-18 19:28:46
categories:
- Github
tags:
- git
---

## 一、 前言
&emsp;最近上传项目的时候，粗心大意～忘记将一些需要忽略的文件夹加进`.gitignore`文件中！！！导致把项目中所有文件都上传到`github.com`上去了，其中有一些文件我并不想上传上去。<!-- more -->所以呢？我现在想要把一个文件夹（例如`bigData`）在`github`删除，但是又不想把本地的`bigData`文件夹删掉～但是！`github`上面只能删除文件（如下图所示），并不支持删除文件夹。


<img src="https://wx1.sinaimg.cn/mw690/e0db46edgy1fqh1t1v3g5j20rk08lmyh.jpg" align="center">

## 二、采用`git`指令来进行删除操作

**代码如下所示：（以删除bigData文件夹为例）**
```
# 首先切换到本地工作目录，也就是与github上对应的那个目录。
cd dir/

# 然后执行下面的git命令
git rm -r --cached bigData    # --cached不会把本地的bigData删除
git commit -m "delete bigData dir"
git push -u origin master

```