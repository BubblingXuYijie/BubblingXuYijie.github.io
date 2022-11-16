---
title: MyBatisPlus 开启事务并交由 SpringBoot 管理
date: 2022-01-09 12:05:14
categories: Java
tags:
    - MyBatisPlus
    - SpringBoot
    - 事务
cover: /img/LinuxLogo.jpg
---
# 前言

<font color=#999AAA >网络上对于事务的解释都太过官方，太过晦涩，导致我们都看不懂，我来用人话解释一下什么是事务。</font>

<font color=#999AAA >而开启事务也很简单，只需要`@EnableTransactionManagement`和`@Transactional`两个注解。</font>



<hr style=" border:solid; width:100px; height:1px;" color=#000000 size=1">


# 一、什么是事务


<font color=#999AAA >假如你的程序正在进行数据库写操作，但是中途程序报错了，所以数据库里被写入了一些残缺的东西；

<font color=#999AAA >如果开启了事务功能，springboot会在程序正常结束后才会写入数据库，如果程序中途报错，事务会回滚，也就是数据库不会被残缺的数据污染

==一句话，不开启事务是实时写入，写入语句执行完数据库就有数据了，开启事务，是所有程序执行完，数据库才有数据。==

程序运行时如果控制台有下面 SqlSession was not registered for synchronization 意思就是事务没有开启。



```bash
Closing non transactional SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@6d8811ab]
Creating a new SqlSession
SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@4dd82b9a] was not registered for synchronization because synchronization is not active
```



# 二、开启方法

<font color=#999AAA> 在启动类上添加一个注解 `@EnableTransactionManagement`，最好自己手打然后回车键自动 import class，复制粘贴的话可能无法识别到这个注解。

![在这里插入图片描述](https://img-blog.csdnimg.cn/3083c337c466404b8fb638559d5a567b.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA57mB5Y2O5bC95aS05ruh5piv5q6H,size_20,color_FFFFFF,t_70,g_se,x_16)

<font color=#999AAA >然后在每个 Service 上都添加一个注解 `@Transactional(rollbackFor = RuntimeException.class)`，代表在 RuntimeException 异常发生时进行回滚，@Transactional里面的参数类型还有很多，有兴趣可以自己学习学习
![在这里插入图片描述](https://img-blog.csdnimg.cn/2ad1344ad981465387f1de32fe2cd31b.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA57mB5Y2O5bC95aS05ruh5piv5q6H,size_19,color_FFFFFF,t_70,g_se,x_16)
<font color=#999AAA >然后查询的时控制台出现以下信息 Registering transaction synchronization for SqlSession ，就是配置成功了

```bash
Registering transaction synchronization for SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@3c34c1ee]
JDBC Connection [HikariProxyConnection@405147465 wrapping com.mysql.jdbc.JDBC4Connection@7e6e118a] will be managed by Spring
Releasing transactional SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@3c34c1ee]
Fetched SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@3c34c1ee] from current transactio
```


<hr style=" border:solid; width:100px; height:1px;" color=#000000 size=1">


# 总结
<font color=#999AAA >具体 MyBatisPlus 的配置和 Druid 或者 Hikari 连接池的配置，看我的传送门[Springboot配置Druid或Hikari连接池（并开启MyBatisPlus事务）](https://blog.csdn.net/qq_48922459/article/details/122051528?spm=1001.2014.3001.5501)

更复杂的 LambdaQurryWrrapper 构造器使用请看[Springboot 使用 Mybatis Plus LambdaQueryWrapper 构造器和注解自定义SQL](https://blog.csdn.net/qq_48922459/article/details/127044399?spm=1001.2014.3001.5502)
