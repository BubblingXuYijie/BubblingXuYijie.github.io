---
title: MySQL数据库创建用户并赋予数据库的操作和登录权限
date: 2021-12-27 17:57:35
categories: 数据库
tags:
    - MySQL
    - 数据库
cover: https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/MysqlLogo.jpg
---
#  前言
<font color=#999AAA >两种方法，第一种方法确保你的账号拥有全部权限，你安装数据库的时候默认的 root 用户就拥有全部权限。</font>

<font color=#999AAA >第二张方法可以是你的账号不具备管理员权限，所以要直接操作 mysql 表，使用 insert 语句添加用户。</font>

<font color=#999AAA >可以设置用户对某个数据库的某个表的增删改查权限，还能限制操作 IP </font>

<hr style=" border:solid; width:100px; height:1px;" color=#000000 size=1">

#  第一种方法

## 一、创建一个新用户 test ，密码设置为 123456</font>

@'%' 的意思是允许 test 用户在任意 IP 登录，如果是 @'localhost'，意思就是只能在本机登录不能远程登录，如果是 @'111.111.111.111'，意思是只能在 IP 为111.111.111.111 的设备上登录
```bash
CREATE USER 'test'@'%' IDENTIFIED BY '123456';
```



## 二、赋予用户权限

###  赋予全部权限

`all`代表全部权限（增删改查），`*.*`  代表全部的数据库的全部表，后面的`@'%'`是在任何 IP 都可以拥有这些权限

```bash
grant all privileges on *.* to 'yj_gongdian'@'%';
```

###  赋予部分表的部分权限

==下面这种写法自定义权限和表==


我先创建个数据库 school，数据库里我又建了个表 students

```bash
create database school;
create table students;
```

这个意思是把 school 这个数据库下面的 students 表的 select 权限赋给 test 用户，而且限制 IP 为111.111.111.111
```bash
grant select on school.students to test@111.111.111.111;
```

<font color=#999AAA >三、刷新权限

必须刷新，不然不生效

```bash
flush privileges;
```
<hr style=" border:solid; width:100px; height:1px;" color=#000000 size=1">


#  第二种方法
<font color=#999AAA >字段意思和上面的一样，意思是在 mysql 数据库里的 user 表，插入一条数据， mysql这个表是自带的，里面存储的都是用户的信息。

```bash
use mysql;
INSERT INTO user(host,user,password,select_priv,insert_priv,update_priv) VALUES('%','test',password('wulw2022!@#'),'y','y','y');
flush privileges;
```

<hr style=" border:solid; width:100px; height:1px;" color=#000000 size=1">

# 总结

