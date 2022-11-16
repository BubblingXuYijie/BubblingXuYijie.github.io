---
title: 使用 JMeter 对 MySQL 数据库进行压力测试
date: 2022-01-20 17:28:29
categories: 运维测试
tags:
    - JMeter
    - MySQL
    - 数据库
    - 运维测试
cover: https://img-blog.csdnimg.cn/8845e12f1f754b6793d9c04400847290.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA57mB5Y2O5bC95aS05ruh5piv5q6H,size_20,color_FFFFFF,t_70,g_se,x_16
---
# 前言

<font color=#999AAA >暂无</font>

<hr style=" border:solid; width:100px; height:1px;" color=#000000 size=1">


# 一、安装并配置 JMeter
##  下载

<font color=#999AAA >[官网下载](https://jmeter.apache.org/download_jmeter.cgi)，下载二进制的这个 zip
![在这里插入图片描述](https://img-blog.csdnimg.cn/8845e12f1f754b6793d9c04400847290.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA57mB5Y2O5bC95aS05ruh5piv5q6H,size_20,color_FFFFFF,t_70,g_se,x_16)
##  配置环境变量

<font color=#999AAA >然后解压到你喜欢的位置，配置环境变量，新建一个 JMETER_HOME

<font color=#999AAA >然后在 path 里添加 `%JMETER_HOME%\bin`

<font color=#999AAA >在 CLASSPATH 的最前部加上
`%JMETER_HOME%\lib\ext\ApacheJMeter_core.jar;%JMETER_HOME%\lib\jorphan.jar;%JMETER_HOME%\lib\logkit-2.0.jar;`

![在这里插入图片描述](https://img-blog.csdnimg.cn/705ce571af0641049321690cb25635c1.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA57mB5Y2O5bC95aS05ruh5piv5q6H,size_20,color_FFFFFF,t_70,g_se,x_16)
##  导入 MySQL 驱动

<font color=#999AAA >把 MySQL 的驱动放进解压的 jmeter 根目录的 lib 文件夹里（这个下载不用教了吧）
![在这里插入图片描述](https://img-blog.csdnimg.cn/477ee88af59d4db09482e71c92801055.png)



<hr style=" border:solid; width:100px; height:1px;" color=#000000 size=1">

# 二、启动 JMeter

<font color=#999AAA >打开命令行，输入`jmeter.bat`就会自动打开 jmeter 的图形界面，建议勾选一下中文

![在这里插入图片描述](https://img-blog.csdnimg.cn/86f11a3d2894433e8d8e61b676ec72a9.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA57mB5Y2O5bC95aS05ruh5piv5q6H,size_20,color_FFFFFF,t_70,g_se,x_16)
![在这里插入图片描述](https://img-blog.csdnimg.cn/8098b7d3e7554a86a656d595515271b3.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA57mB5Y2O5bC95aS05ruh5piv5q6H,size_18,color_FFFFFF,t_70,g_se,x_16)
<font color=#999AAA >点击下面的浏览，找到你的 lib 里的驱动，双击一下

![在这里插入图片描述](https://img-blog.csdnimg.cn/cfc373dd3646441aaa3ad6351b5f887c.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA57mB5Y2O5bC95aS05ruh5piv5q6H,size_20,color_FFFFFF,t_70,g_se,x_16)

<hr style=" border:solid; width:100px; height:1px;" color=#000000 size=1">


#  三、开始进行压力测试

##  配置

<font color=#999AAA >右键 TestPlan 新建 线程组
![在这里插入图片描述](https://img-blog.csdnimg.cn/abd37794e02045a5882ef4e9472ffa06.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA57mB5Y2O5bC95aS05ruh5piv5q6H,size_13,color_FFFFFF,t_70,g_se,x_16)

<font color=#999AAA >线程数就是模拟的用户数， Ramp-Up时间 是指用户在多久时间内请求完毕，
下面的意思是 在 1 秒内，100 个用户同时请求数据库，循环次数代表一共执行 5 次。建议线程数设置大一点，效果好。

![在这里插入图片描述](https://img-blog.csdnimg.cn/e0b5ffd15d8c47d49d6691e525c2e4a5.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA57mB5Y2O5bC95aS05ruh5piv5q6H,size_20,color_FFFFFF,t_70,g_se,x_16)

<font color=#999AAA >右键线程组，添加一个 JDBC 的配置元件，里面只需要配置下面框选的

![在这里插入图片描述](https://img-blog.csdnimg.cn/e1a872ac00b347e191accac2a0930f59.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA57mB5Y2O5bC95aS05ruh5piv5q6H,size_20,color_FFFFFF,t_70,g_se,x_16)
![在这里插入图片描述](https://img-blog.csdnimg.cn/0d11ed7b432d42f7aa155b3d4a1a004a.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA57mB5Y2O5bC95aS05ruh5piv5q6H,size_20,color_FFFFFF,t_70,g_se,x_16)


<font color=#999AAA >右键 线程组，添加一个 JDBC Request 取样器，test 是前面配置的 pool name，框里写 SQL 语句，类型可以选查询或修改

![在这里插入图片描述](https://img-blog.csdnimg.cn/6e3826a1daa74736b7f5a1b63d1dd865.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA57mB5Y2O5bC95aS05ruh5piv5q6H,size_19,color_FFFFFF,t_70,g_se,x_16)
![在这里插入图片描述](https://img-blog.csdnimg.cn/302a8f7e41544cdfbb60cfb67122de71.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA57mB5Y2O5bC95aS05ruh5piv5q6H,size_20,color_FFFFFF,t_70,g_se,x_16)

<font color=#999AAA >右键 线程组，添加几个监听器，这是看测试结果的东西，你们可以多加点自己玩玩。

![在这里插入图片描述](https://img-blog.csdnimg.cn/9b99f2ee07004b23a56a82a1afa0c1c5.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA57mB5Y2O5bC95aS05ruh5piv5q6H,size_20,color_FFFFFF,t_70,g_se,x_16)


##  观察结果

<font color=#999AAA >如果 结果树 里面大多都是报错，说明 线程数 设置的太大了，也就是你的数据库承受不起。

![在这里插入图片描述](https://img-blog.csdnimg.cn/2f40537ef3664a1a98c11a14bcb64070.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA57mB5Y2O5bC95aS05ruh5piv5q6H,size_20,color_FFFFFF,t_70,g_se,x_16)
![在这里插入图片描述](https://img-blog.csdnimg.cn/db0fef2163c64403b63f0d7dcc53ed1a.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA57mB5Y2O5bC95aS05ruh5piv5q6H,size_20,color_FFFFFF,t_70,g_se,x_16)


<hr style=" border:solid; width:100px; height:1px;" color=#000000 size=1">

# 总结

`样本数目 ：`是指在测试过程中，总共向服务器发出的请求数目。成功的情况下等于你设定的并发数目 ×   循环次数
`最大值：`响应时间的最大值
`吞吐量 ：` 表示服务器每分钟处理的请求数目。
`平均值 ：` 总的运行时间除以发送到服务器的请求数目；
`偏离 ：` 服务器响应时间变化、离散程度测量值的大小，或者，换句话说，就是数据的分布。
`中位数 :`  时间的数字，有一半的服务器响应时间低于该值而另一半高于该值。
`异常 :`  样本接收失败率 
