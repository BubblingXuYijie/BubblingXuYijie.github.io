---
title: Linux 使用 apt 安装 zookeeper
date: 2022-09-29 14:37:45
categories: Linux
tags:
    - Linux
    - apt
    - zookeeper
cover: https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/zookeeper.jpg
---
# 前言

`zookeeper`：是一个微服务注册中心，将一个个 Java 项目注册到 zookeeper，然后使用 openfeign 或者 dubbo 就可以实现这些 Java 项目之间的互相调用。

> 相较于去下载 zookeeper.tar 压缩包的方式来安装，使用 apt 安装的好处就是，方便，只需一个命令，而且安装以后无需配置环境变量

`本教程用 Debian11 来演示，适用于 Ubuntu 系的全部系统，CentOS 系统将 apt 命令换为 yum 即可，或者 `yum install apt`就可以使用 apt 了`

---



# 一、安装
> 首先你的 Linux 要有 jdk 环境，没有的要装一下，还是一样，一句命令，无需配置环境变量
> 注意：Debian11 的仓库只有 jdk 11 和 17，所以想安装 1.8 需要手动下载压缩包，配置环境变量，11 和 17 并不影响 zookeeper 的正常启动，只是有些小瑕疵

```bash
apt install openjdk-17-jdk
```

```bash
apt install zookeeper
```
`安装完成后，zookeeper 的配置文件在 /etc/zookeeper/conf 这个文件夹下的 zoo.cfg ，安装完成后不需要修改配置文件即可使用`
```bash
root@debian:/usr/share/zookeeper/bin# cd /etc/zookeeper/conf
root@debian:/etc/zookeeper/conf# ls
configuration.xsl  environment	log4j.properties  myid	zoo.cfg
root@debian:/etc/zookeeper/conf# vim zoo.cfg 
```

> 开放防火墙 2181 端口

```bash
# Debian/Ubuntu ufw
ufw allow 2181
ufw reload
# Debian/Ubuntu iptables（这个叼毛防火墙好麻烦，我没用过，不知道是不是这样）
iptables -A INPUT -p tcp --dport 2181 -j ACCEPT
iptables-restore
# CentOS
firewall-cmd --zone=public --add-port=2181/tcp --permanent
firewall-cmd --reload
```

---

# 二、启动

`zookeeper 的启动脚本在 /usr/share/zookeeper/bin 这个文件夹下的 zkServer.sh ，我们来启动，启动之前请确保你对这个文件有足够权限，如果提示没有权限，运行` chmod 775 zkServer.sh `即可`

```bash
cd /usr/share/zookeeper/bin/
sh zkServer.sh start
# 停止命令：sh zkServer.sh stop
# 重启命令：sh zkServer.sh restart
# 查看状态: sh zkServer.sh status
```
==注意：如果你是高版本的 Ubuntu 或者 Debian 系统（比如我的 Debian11），运行 zkServer 肯定会报下面这个错，因为我们的系统执行脚本默认的使用的是 dash，zookeeper用的是bash，所以要运行一下这个命令再启动:`dpkg-reconfigure dash`==

```bash
zkServer.sh: 157: zkServer.sh: Syntax error: "(" unexpected (expecting ";;")
原因是：zookeeper使用的shell版本和系统使用的shell版本不兼容，当前ubuntu系统的shell默认使用的是dash,而zookeeper使用的是bash
```

`连接 zookeeper ，下面这样就是成功了`

```bash
root@debian:/usr/share/zookeeper/bin# sh zkCli.sh -server localhost:2181
Connecting to localhost:2181
Welcome to ZooKeeper!
JLine support is enabled
[zk: localhost:2181(CONNECTING) 0] 

```

> 上面我说 jdk 17 运行 zookeeper 有些小瑕疵，瑕疵就是：ls / 查看注册的服务节点时会报错，后面可能还会有其他命令报错，不过不影响正常使用，我已经用程序测试了，zookeeper 一切正常，只是zookeeper控制台命令有问题
> ![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/zookeeperInstall.png)



---



# 总结
