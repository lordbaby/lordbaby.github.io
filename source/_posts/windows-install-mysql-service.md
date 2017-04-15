---
title: windows下安装MySQL服务
date: 2017-04-15 21:33:15
tags: [MySQL]
categories: MySQL
---

这种教程网上一搜一大堆，期间遇到一坑，就是my.ini文件中有汉字注释的话，必须另存为是ANSI格式文件，这里也就以文字形式简要记录一下安装步骤。
<!--more-->

# 下载MySQL

地址：http://dev.mysql.com/downloads/mysql/
主要几个坑 ：

1. 不要条件反射似的被注册和登录后才能下载吓到，也许只有国内某些网站才能干的出来。点击左下角的**No thanks,just start my downlload.**

# 添加环境变量

添加环境变量主要是使用命令时，省去命令前长长的路径，这里把MySQL目录下的bin目录添加到系统变量的path下面。

# 配置mysql的配置文件

1. 新建my.ini 
2. 记事本打开，另存为ANSI格式文件
```
[mysqld]

basedir=E:\mysql\mysql-5.7.18-winx64

datadir=E:\mysql\mysql-5.7.18-winx64\data

port=3306

character-set-server=utf8

max_connections=200

sql-mode=STRICT_TRANS_TABLES,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION

default-storage-engine=MYISAM

skip-grant-tables

[client]

port=3306

default-character-set=utf8

```

# 安装MySQL

1. 以管理员身份启动CMD
2. `mysqld -install` 安装MySQL服务
3. `mysqld --initialize` 初始化data数据
4. `net start mysql` 启动MySQL服务
5. OK貌似一切顺利，只知道用户名是root，初始密码还不知。接下来
6. 在配置文件mysqld下面插入skip-grant-tables,跳过验证
7. 重启MySQL服务（改配置文件了就得重启，这个规定在编程界通用）
8. `mysql -uroot -p`, 遇见输入密码直接回车跳过
9. 重置root密码，`update mysql.user set authentication_string=password('123456') where user='root' and Host = 'localhost';`

OK一切顺利。

