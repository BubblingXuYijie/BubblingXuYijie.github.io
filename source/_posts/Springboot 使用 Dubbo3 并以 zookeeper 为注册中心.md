---
title: Springboot 使用 Dubbo3 并以 zookeeper 为注册中心
date: 2022-09-29 15:22:48
categories: Java
tags:
    - SpringBoot
    - Dubbo
    - zookeeper
    - 微服务
    - RPC
cover: https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/Dubbo.jpg
---
# 前言
> `dubbo`：简单来讲就是一个 RPC 调用框架，类似于 SpringCloud + OpenFeign，支持 nacos、zookeeper 等注册中心，拥有图形界面，可使用界面管理 zookeeper 的节点信息
> `zookeeper`：是一个微服务注册中心，将一个个 Java 项目注册到 zookeeper，然后使用 openfeign 或者 dubbo 就可以实现这些 Java 项目之间的互相调用

---

`zookeeper 可以连接线上的，也可以安装在本地，windows 安装很简单，你们自己去搜，Linux 安装我这里还有另一篇文章，一个 apt 命令的事情，无需配置环境变量什么的`

[Linux 使用 apt 安装 zookeeper](https://blog.csdn.net/qq_48922459/article/details/127105615?spm=1001.2014.3001.5502)

# 一、构建微服务项目
==一定要选择 Springboot 3.0 以下的版本，和 jdk 14以下的版本，高于等于3.0和14 dubbo3 和 zookeeper 支持不好==
## 1、新建项目

> 最基础的结构，构建方法是，先 new 一个 DubboDemo 的 Springboot 项目，不需要勾选任何依赖

![在这里插入图片描述](https://img-blog.csdnimg.cn/0e48521c5baf411d821d798051ffdf4b.png)
> 然后右键 DubboDemo ，new 一个 module，也选择 Springboot 项目，勾选一个 web 依赖即可，把 api 、zookeeperConsumer、zookeeperProvider都 new 出来
>
![在这里插入图片描述](https://img-blog.csdnimg.cn/dcb63b1ada04451ca977fccf286a2c38.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/2b76e66eb101402fa1c8d924bdcdcc9b.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/520b26889ba34339be17b1b4f739bd01.png)
## 2、修改父POM

> 也就是把 DubboDemo 的 POM的 dependency 全部删掉，添加`<packaging>pom</packaging>`和`<modules></modules>`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.7.4</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>icu.xuyijie</groupId>
    <artifactId>DubboDemo</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>DubboDemo</name>
    <description>DubboDemo</description>
    <packaging>pom</packaging>
    <modules>
        <module>zookeeperConsumer</module>
        <module>zookeeperProvider</module>
        <module>api</module>
    </modules>
    <properties>
        <java.version>1.8</java.version>
    </properties>
</project>

```

## 3、修改子POM
> 把子模块的 POM 中的 `<parent></parent>`都换成下面这个就行了，注意 `icu.xuyijie`是我的包名，以你们的项目为准，这样一个基础的微服务项目就搭建好了

```xml
<parent>
    <groupId>icu.xuyijie</groupId>
    <artifactId>DubboDemo</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <relativePath/> <!-- lookup parent from repository -->
</parent>
```
---

# 二、写代码

## 1、引入依赖
> 在 zookeeperCondumer 和 zookeeperProvider 这两个模块的 pom 里引入下面两个依赖

```xml
<dependency>
    <groupId>org.apache.dubbo</groupId>
    <artifactId>dubbo-spring-boot-starter</artifactId>
    <version>3.1.0</version>
</dependency>
<dependency>
    <groupId>org.apache.dubbo</groupId>
    <artifactId>dubbo-dependencies-zookeeper</artifactId>
    <type>pom</type>
    <version>3.1.0</version>
</dependency>
```


## 2、api 模块

> 这个模块什么都不用改，直接新建一个 Service 里面写个方法就行了
>
![在这里插入图片描述](https://img-blog.csdnimg.cn/a21b0c71bd5a447e80849aacfb732e23.png)

## 3、zookeeperProvider 模块
> 在 pom 里添加一下对 api 模块的依赖，这样我们就可以注入 api 模块里的 service 了，zookeeperConsumer 的 pom 也要引入 api 模块

![在这里插入图片描述](https://img-blog.csdnimg.cn/b9174f34f379419b93a040f5bfddd811.png)
> 修改 yml 配置文件

```yaml
server:
  port: 8081
dubbo:
  application:
    # 注册到 zookeeper 里面后显示的服务名称
    name: dubbo-springboot-demo-provider
  # 远程调用协议
  protocol:
    name: dubbo
    port: -1
  # 连接 zookeeper
  registry:
    id: zk-registry
    address: zookeeper://192.168.0.105:2181
    timeout: 60000
    parameters:
      blockUntilConnectedWait: 250
  config-center:
    address: zookeeper://192.168.0.105:2181
  metadata-report:
    address: zookeeper://192.168.0.105:2181
```

> 写 ServiceImpl ，实现方法，看见没有，我们引入的 api 模块，可以直接注入到我们的 provider 模块中，记得要用 `@DubboService` 替换@Service注解，这个注解代表我们的这个 impl 是一个远程服务提供者

![在这里插入图片描述](https://img-blog.csdnimg.cn/649ff9e7e0fe4c74a10abc62efda8ec1.png)
> 启动类增加`@EnableDubbo`注解

![在这里插入图片描述](https://img-blog.csdnimg.cn/3d8cef27d8ff4f758d404c67d34484f8.png)

## 4、zookeeperConsumer 模块

> 引入 api 模块依赖，启动类增加`@EnableDubbo`注解，然后 yml 配置文件和 provider 一样，就是需要改个 application-name，和 provider 的不一样就可以了

![在这里插入图片描述](https://img-blog.csdnimg.cn/052fe45bbb5147408b267ebbbcc2377e.png)

> 然后代码只需要写一个 controller 就行了，注入 api 模块的 Service，用`@DubboReference`注解，这个注解代表这个 service 是一个远程服务，需要使用 dubbo 的 RPC 进行调用

![在这里插入图片描述](https://img-blog.csdnimg.cn/6c15a7a755d842e49348fd0452f4f3ad.png)






---

# 三、开始测试
> 启动 zookeeperProvider 和 zookeeperConsumer 两个模块

![在这里插入图片描述](https://img-blog.csdnimg.cn/16f0505bec1a43f381474a02d8b17054.png)
> 浏览器请求我们的 zookeeperConsumer 的 controller

![在这里插入图片描述](https://img-blog.csdnimg.cn/fd1e4175b6ff4b39bfd5fa597e342d9d.png)

> 发现在 zookeeperProvider 的控制台中打印了下面的文字，说明远程调用成功

![在这里插入图片描述](https://img-blog.csdnimg.cn/0b40e7c508814e6c9b2eeae067a32b1e.png)




---

# 总结
我的本地没有安装 Dubbo 的图形界面，所以没法给你们展示注册的服务信息，后续会写 dubbo + nacos 注册中心，nacos 自带图形界面，到时再仔细唠唠
