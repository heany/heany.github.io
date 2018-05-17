---
title: Nginx+PHP+MySQL搭建过程
date: 2018-05-15 16:43:10
categories:
- Website
tags:
- Nginx-php-MySQL 
---

## 一、 前言
&emsp;最近要开始干活了～老板要求边做边写开发文档，啊啊啊啊，就把这一系列的文档发在博客里面吧，需要的时候再来拿，还能凑几篇博客。这是第一篇，关于Nginx+php+MySQL环境搭建的，很简单网上很多～～～
<!-- more -->

## 二、 本机环境

+ OS: Ubuntu 17.10.1_x64
+ Nginx_version: nginx/1.12.1
+ PHP_version: PHP-7.1.17
+ MySQL_version: mysql5.7.22

## 三、 安装步骤
### 3.1、 安装`Nginx Web Server`
直接使用`apt package management`来完成安装，命令如下

```
sudo apt-get update
sudo apt-get upgrade
sudo apt-get install nginx

# 等一会儿就安装好了，下面是一些重要的文件目录
nginx的配置文件目录： /etc/nginx;
Nginx的缓存目录： /var/cache/nginx;
Nginx的日志目录： /var/log/nginx;
```

>    # 启动nginx服务
>    
>    sudo systemctl start nginx
>    
>    # test
>    
>    #打开网页，输入`http://localhost`可以看到`welcome to nginx`字样

### 3.2、 安装MySQL管理网站数据
同样是使用`ubuntu`中的`apt`包管理器来安装，命令如下：
```
sudo apt-get install mysql-server
# 会弹框提醒输入root密码，直接输入然后按`Enter`，结束。
```

### 3.3、 安装`PHP`服务
同样是使用`ubuntu`中的`apt`包管理器来安装，命令如下：
```
# 顺便把php-mysql插件装上
sudo apt-get install php-fpm php-mysql

# 一些重要的文件目录
# php配置文件路径： /etc/php/7.1/fpm/php.ini
```

然后，修改`php`的配置文件：
```
# 切换到php配置目录
cd /etc/php/7.1/fpm
# 修改配置文件
sudo vim php.ini
++++
找到这行，取消注释，将1改成0
cgi.fix_pathinfo=0
++++
```

最后，重启`php`服务
```
sudo systemctl restart php-7.1-fpm
```

### 3.4、 配置`Nginx`去使用`PHP`处理器
```
# 切换到nginx网站的配置目录
cd /etc/nginx/sites-available
sudo vim default
# 做如下修改：
++++
server {
    listen 80 default_server;
    listen [::]:80 default_server;

    root /var/www/html;
    index index.php index.html index.htm index.nginx-debian.html;

    server_name server_domain_or_IP;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php7.1-fpm.sock;
    }

    location ~ /\.ht {
        deny all;
    }
}
++++
```

<b style="color: red">这里有个小坑，了解一下：</b><b>由于安装的php版本是`php7.1`，安装`nginx`默认配置文件里面使用的是`/run/php/php7.0-fpm.sock`,你在`/run/php`中找不到`php7.0`的sock文件。所以，这里我们要把它修改为我们安装的`php`相应版本的`sock`文件。即</b><b style="color: red">`/run/php/php7.1-fpm.sock;`</b>

然后，测试配置文件是否生效。

    sudo nginx -t  #如果没报错，说明配置成功


最后，重新加载`nginx`服务器

    sudo systemctl reload nginx

### 3.5、 测试`nginx`和`PHP`配置是否生效
在网站的根目录`/var/www/html`下新建一个`info.php`文件，写入一下内容：
``` php
// info.php
<?php
phpinfo();
?>
```

然后，打开浏览器，输入`localhost/info.php`，会看到输出了之前安装的`php`的版本信息和其他模块的详细信息。这 就 表 明 `Nginx` 和`PHP`都安装和配置～～成功了！！！

## 四、 结束语
路很长～这才刚刚开始啊～～～～

[参考自博客](https://www.digitalocean.com/community/tutorials/how-to-install-linux-nginx-mysql-php-lemp-stack-in-ubuntu-16-04)