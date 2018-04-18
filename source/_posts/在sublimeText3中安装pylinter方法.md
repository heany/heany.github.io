---
title: 在sublimeText3中安装pylinter方法
date: 2017-10-15 14:03:54
categories:
- 常用工具
tags:
- sublime 插件
comments: true
---


## 1.下载pylint安装包
    官网下载，解压即可<https://pypi.python.org/pypi/pylint>
    
## 2.安装 astroid 
    pip install astroid
或者
   &nbsp;&nbsp;&nbsp;&nbsp;打开<https://bitbucket.org/logilab/astroid>,下载压缩包，解压 cd到下载好的文件夹内，然后使用

    python setup.py install
    
## 3.安装isort,用如下命令
    pip install isort
    
## 4.安装pylint
   &nbsp;&nbsp;&nbsp;&nbsp;cmd切换到刚才下载解压pylint文件夹内，输入命令：

       python setup.py install
       
## 5.安装pylint插件
>在sublime中ctrl+shift+P 然后输入install 然后输入pylinter，回车即可！
