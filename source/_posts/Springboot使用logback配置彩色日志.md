---
title: Springboot使用logback配置彩色日志
date: 2024-04-18 19:18:01
categories:
  - Java
tags:
  - SpringBoot
  - logback
cover: https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/彩色日志.png
---

# 前言

> 应该有很多同学发现，使用了`logback`以后，我们的控制台日志都变成灰色了，网络上搜到的logback配置大多数没有进行配色，所以会把springboot的默认配色方案给覆盖掉

![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/黑白日志.png)


---

# 一、logback文件

> 重点对比`<property name="nocolor.log.pattern">`和`<property name="log.pattern>"`这两行，我们用类似`%blue()`这种颜色把日志内容包裹起来，就可以了

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration scan="true" scanPeriod="60 seconds">
	 <!-- 无配色 -->
	<property name="nocolor.log.pattern" value="%d{yyyy-MM-dd HH:mm:ss.SSS} %-5level [%thread] %logger{36} - [%M:%L] - %msg%n" />
    <!-- 有配色 -->
    <property name="log.pattern" value="%blue(%d{yyyy-MM-dd HH:mm:ss.SSS}) %highlight(%-5level) %magenta([%thread]) %cyan(%logger{36} - [%M:%L]) - %msg%n"/>

    <!-- 控制台输出 -->
    <appender name="console" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>${log.pattern}</pattern>
        </encoder>
    </appender>

    <root level="info">
        <appender-ref ref="console"/>
    </root>
</configuration>
```


---
# 二、效果

![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/彩色日志.png)
