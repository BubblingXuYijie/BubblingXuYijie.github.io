---
title: 关于 AOP 切面导致 WebSocket 的 @ServerEndPoint 无法注入的问题
date: 2022-01-24 13:13:55
categories: Java
tags:
    - WebSocket
    - SpringBoot
    - AOP
cover: /img/springbootLogo.jpeg
---
# 前言

<font color=#999AAA >今天给我的毕业设计加上了 `AOP` 日志拦截，结果导致了 `WebSocket` 的报错。，错误信息为：</font>

```bash
Failed to register @ServerEndpoint class: class pers.xuyijie.communityinteractionsystem.websocket.MyWebSocket$$EnhancerBySpringCGLIB
Caused by: javax.websocket.DeploymentException: Cannot deploy POJO class [pers.xuyijie.communityinteractionsystem.websocket.MyWebSocket$$EnhancerBySpringCGLIB
```

<hr style=" border:solid; width:100px; height:1px;" color=#000000 size=1">


# 原因和解决方案

<font color=#999AAA >原因是因为 `WebServerContainer` 里面的 `addEndPoint` 方法里的 `annotation` 为 null。![在这里插入图片描述](https://img-blog.csdnimg.cn/c3191d4ed9844f3d91c99bca7fa9ad57.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA57mB5Y2O5bC95aS05ruh5piv5q6H,size_20,color_FFFFFF,t_70,g_se,x_16)<font color=#999AAA >感谢 华阳余文乐https://blog.csdn.net/qq_15807785/article/details/83547978 ，上图是他的调试截图。

<font color=#999AAA >导致为 null 的原因是因为 aop 的 `@PointCut` 注解和 Aop 自定义的 `@Log` 注解，总之，aop 不能作用到 websocket 的文件，否则就会导致问题。

![在这里插入图片描述](https://img-blog.csdnimg.cn/361908975fad453e9da976b281d113ec.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/bd9943208a464ff18ac3cb4751503558.png)
<font color=#999AAA >去掉 WebSocket 上的 `@Log`，并确保 `@PointCut 的 execution 中的包路径不包含` WebSocket 的文件就可以了。

启动成功
![在这里插入图片描述](https://img-blog.csdnimg.cn/f56aad6ac3ba4ae6abf391cd1b074fcc.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA57mB5Y2O5bC95aS05ruh5piv5q6H,size_20,color_FFFFFF,t_70,g_se,x_16)


<hr style=" border:solid; width:100px; height:1px;" color=#000000 size=1">
