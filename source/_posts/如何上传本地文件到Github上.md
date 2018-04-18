---
title: 如何上传本地文件到Github上
date: 2017-10-15 21:14:57
categories:
    - Github
tags:

---

<!-->
    最近做了一些工作，想将他们上传到GitHub上去保存，我习惯于本地编辑，完成后再一起上传到仓库中，长时间不使用很容易忘，总结网上的一些教程，写作博客里方便自己偶尔查阅。
<!-- more -->

本文参考以下博文
[两种方法上传本地文件到github](http://www.jianshu.com/p/c70ca3a02087)

想知道详细的步骤可以点击前往学习，这里我只简单记录一些常用的命令。

#### 1.GitHub在线上传文件夹

##### 1.1点击`Upload files`

##### 1.2 直接拖拽



#### 2.通过git工具上传本地文件夹

##### 2.1下载git
>  由于部分原因，国内直接官网下载git非常慢，需要FQ。这里提供一个国内的下载站，方便网友下载。

[**点击前往**](https://github.com/waylau/git-for-win)

##### 2.2 绑定用户
打开git-bash.exe（直接在桌面上点击右键，或者点击开始按钮找到Git Bash）

在打开的GIt Bash中输入以下命令（用户和邮箱为你github注册的账号和邮箱）

    $ git config --global user.name "##"
    $ git config --global user.email "h##@**.com"

##### 2.3 设置SSH Key

###### 2.3.1 生成ssh key

###### 2.3.2 为GitHub账号配置ssh key

##### 2.4 上传本地项目到GitHub
> **这里只贴一些命令**

```
# 初始化
git init

# 将所有文件添加到仓库中
git add .

# 提交
git commit -m "注释"

# 关联GitHub仓库
git remote add origin https://github.com/**/##.git

# 上传本地代码及文件
git push -u origin master


```
