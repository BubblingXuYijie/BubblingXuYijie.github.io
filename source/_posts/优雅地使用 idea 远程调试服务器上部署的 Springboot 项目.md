---
title: 优雅地使用 idea 远程调试服务器上部署的 Springboot 项目
date: 2022-01-18 13:41:53
categories: Java
tags:
    - Java
    - SpringBoot
    - idea
    - 线上调试
cover: https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/远程调试0.png
---
# 前言

<font color=#999AAA >听起来好像没有什么用，为什么不在本地调试？因为有一些项目功能需要部署在线上才能正常使用，比如有些接口只能通过服务器来进行访问，而本地是不能访问的，所以要在服务器上进行调试。</font>

<hr style=" border:solid; width:100px; height:1px;" color=#000000 size=1">


# 一、启动服务器上的项目

<font color=#999AAA >为了简单我们直接 jar 方式启动，`-Xdebug -Xrunjdwp:transport=dt_socket,server=y,suspend=n,address=10000`就是远程调试需要的配置，`address` 设置为和你的项目不同的端口，`XXX.jar` 是你的 jar 包名字。

```bash
nohup java -jar -Xdebug -Xrunjdwp:transport=dt_socket,server=y,suspend=n,address=10000 XXX.jar > system.log 2>&1 &
```


# 二、配置 idea


<font color=#999AAA >Edit Configurations，添加一个 Remote JVM Debug，写上你的服务器 ip 和刚刚启动的 adress 端口----10000
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/远程调试1.png)

![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/远程调试0.png)

<font color=#999AAA >然后点击 Debug ，控制台出现下方即为成功

![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/远程调试2.png)

<hr style=" border:solid; width:100px; height:1px;" color=#000000 size=1">

# 三、开始调试
<font color=#999AAA >直接在 idea 的代码上打断点（当然你的代码要和服务器上的一致），调用服务器接口就行，从前端直接操作也行，用 Postman 也行，后面就和普通调试一样了。

<font color=#999AAA >别搞混了，调用的端口还是你原本项目的端口，不是 10000 这个端口

