---
title: hexo常用命令
date: 2017-10-28 14:18:08
categories:
    - Website
tags:
    - hexo常用命令
---


现在来介绍常用的Hexo 命令

    npm install hexo -g #安装Hexo
    npm update hexo -g #升级 
    hexo init #初始化博客

命令简写

    hexo n "我的博客" == hexo new "我的博客" #新建文章
    hexo g == hexo generate #生成
    hexo s == hexo server #启动服务预览
    hexo d == hexo deploy #部署

    hexo server #Hexo会监视文件变动并自动更新，无须重启服务器
    hexo server -s #静态模式
    hexo server -p 5000 #更改端口
    hexo server -i 192.168.1.1 #自定义 IP
    hexo clean #清除缓存，若是网页正常情况下可以忽略这条命令

>  博客搭建好以后，发表文章步骤及命令

> * step1：切换到工作目录，执行`hexo n "文章的title"` , 然后看到`source/_posts`会出现刚刚新建的`文章的title.md`
> * step2：进去编辑刚刚生成的md文件
> * step3：执行命令`hexo clean`
> * step4: 执行命令`hexo g`
> * step5: 执行命令`hexo d`等待一会就能看到博客主页出现了刚刚编写的文章


卸载hexo：

    npm uninstall hexo-cli -g #3.0.0版本执行
    npm uninstall hexo -g # 之前版本执行