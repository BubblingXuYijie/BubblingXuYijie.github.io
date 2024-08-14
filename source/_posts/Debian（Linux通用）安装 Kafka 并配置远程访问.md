---
title: Debian（Linux通用）安装 Kafka 并配置远程访问
date: 2022-11-01 15:22:22
categories: Linux
tags:
    - Debian
    - Linux
    - Kafka
cover: https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/Kafka.jpg
---
# 前言
> As we all know，当今世界最流行的消息中间件有 RabbitMq、RocketMq、Kafka，其中，应用最广泛的是 `RabbitMq`，`RocketMq` 是阿里巴巴的产品，性能超过 RabbitMq，已经经受了多年的双11考验，但是怕哪天阿里不维护了，用的人不多，`Kafka` 是吞吐量最大的一个，远超前两个，支持事务、可保证消息的不丢失（网上说的事务和消息可靠性不支持是说的旧版，2以后就开始支持了），对比来讲，Kafka相对于前两个，只有一个劣势，不太支持延时队列，其他方面都要优于它们（个人使用体验，勿喷）。

---


# 一、下载
> 为 Kafka 创建一个安装文件夹，你喜欢哪就装哪

```bash
cd /
mkdir data
cd data/
mkdir kafka
cd kafka/
```

> 下载官方安装包

```bash
# 下载官方安装包，apache大家都知道，下载很慢，大家可以从镜像下载或者挂梯子下载完传输到服务器
wget https://dlcdn.apache.org/kafka/3.3.1/kafka_2.13-3.3.1.tgz
```
`apache大家都知道，下载很慢，下面我的服务器下载速度只有96.8KB/s，大家可以从镜像下载或者挂梯子下载完传输到服务器`

```bash
root@VM-12-15-debian:/data/kafka# wget https://dlcdn.apache.org/kafka/3.3.1/kafka_2.13-3.3.1.tgz
--2022-11-01 14:50:29--  https://dlcdn.apache.org/kafka/3.3.1/kafka_2.13-3.3.1.tgz
Resolving dlcdn.apache.org (dlcdn.apache.org)... 151.101.2.132, 2a04:4e42::644
Connecting to dlcdn.apache.org (dlcdn.apache.org)|151.101.2.132|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 105053134 (100M) [application/x-gzip]
Saving to: ‘kafka_2.13-3.3.1.tgz’

kafka_2.13-3.3.1.tgz            2%[>            ]   2.71M  96.8KB/s    eta 7m 54s
```



# 二、安装

> 在我们创建的文件夹里解压

```bash
# 解压
tar -xzf kafka_2.13-3.3.1.tgz
# 进入解压出来的文件夹
cd kafka_2.13-3.3.1/
```
`解压完成，进入解压的文件夹，ls，出现下面这样几个目录`

```bash
root@VM-12-15-debian:/data/kafka# ls
kafka_2.13-3.3.1.tgz
root@VM-12-15-debian:/data/kafka# tar -xzf kafka_2.13-3.3.1.tgz
root@VM-12-15-debian:/data/kafka# ls
kafka_2.13-3.3.1  kafka_2.13-3.3.1.tgz
root@VM-12-15-debian:/data/kafka# cd kafka_2.13-3.3.1/
root@VM-12-15-debian:/data/kafka/kafka_2.13-3.3.1# ls
bin  config  libs  LICENSE  licenses  NOTICE  site-docs
root@VM-12-15-debian:/data/kafka/kafka_2.13-3.3.1#
```

---


# 三、配置远程访问
> `注意：新版的 Kafka 已经可以不依赖并且不建议依赖 zookeeper 来启动了`，所以我们采用 Kafka 内置的 `KRaft` 启动方式，无需额外安装其他软件，所以我们修改的配置文件的路径如下

```bash
# 修改 kraft 里面的配置文件
vim config/kraft/server.properties
```

`把 advertised.listeners 的 localhost 修改为当前服务器的公网 IP，如果没有公网 IP 就写 192.168 这样的 IP 即可`

> 修改前

![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/DebianKafka0.png)
> 修改后

![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/DebianKafka1.png)
`修改完成保存`

> 开放防火墙 9092 端口

```bash
# Debian/Ubuntu ufw
ufw allow 9092
ufw reload
# Debian/Ubuntu iptables（这个叼毛防火墙好麻烦，我没用过，不知道是不是这样）
iptables -A INPUT -p tcp --dport 9092 -j ACCEPT
iptables-restore
# CentOS
firewall-cmd --zone=public --add-port=9092/tcp --permanent
firewall-cmd --reload
```

---


# 四、启动
> 下面是启动命令，要进入解压的文件夹里面执行哦，格式化 kraft 文件夹命令新安装后只需执行一次，后续启动就不需要了

```bash
# 格式化 kraft 文件夹（新安装后只需执行一次）
KAFKA_CLUSTER_ID="$(bin/kafka-storage.sh random-uuid)"
bin/kafka-storage.sh format -t $KAFKA_CLUSTER_ID -c config/kraft/server.properties
# 启动
bin/kafka-server-start.sh -daemon config/kraft/server.properties &
```
`出现下面这样，启动成功，如果大家想看启动日志，把上面命令的最后一个 & 去掉就可以`

```bash
root@VM-12-15-debian:/data/kafka/kafka_2.13-3.3.1# ls
bin  config  libs  LICENSE  licenses  NOTICE  site-docs
root@VM-12-15-debian:/data/kafka/kafka_2.13-3.3.1# bin/kafka-server-start.sh -daemon config/kraft/server.properties &
[1] 3783821
```

---

# 总结
Springboot 集成 Kafka 的配置和使用，看我的另一篇[Springboot 配置使用 Kafka](https://blog.csdn.net/qq_48922459/article/details/127634598?spm=1001.2014.3001.5501)，不多BB，不会给你扯原理，只会教你怎么用，详细但不啰嗦，你不会后悔的
