---
title: Linux使用Docker安装Nacos并配置MySQL数据源，将Springboot的配置文件部署到Nacos
date: 2021-12-24 09:21:56
categories: Linux
tags:
    - Linux
    - SpringBoot
    - Nacos
    - Docker
cover: /img/NacosLogo.png
---
# 前言

> 为什么要把配置文件放到 Nacos 上
> 1、 采用本地配置，不方便查看当前项目配置，也不方便修改（要重新打包重启），在 Nacos 上可以方便地查看和修改
> 2、易引发生产事故/方便开发测试：比如在发布的时候，容易将测试环境的配置带到生产上，引发生产事故，而项目的启动脚本可以指定 Nacos 上面的配置文件，从而使测试配置文件失效，所以开发的时候无需把精力放在修改配置文件上


---



# 一、Docker中安装配置Nacos


<font color=#999AAA >当然如果不安装在 Docker 里也行，不安装在 Docker 把在 Nacos 的官网下载文件，直接解压就行，跳过 docker pull 和 run 的过程就可以了，然后你要把 nacos 注册为服务，开机启动，然后其他步骤都一样，总之不安装在 Docker 有点麻烦的。

##  安装Docker
<font color=#999AAA >所有 Linux 系统通用安装命令
```bash
curl -sSL https://get.daocloud.io/docker | sh
```

##  拉取Nacos镜像
<font color=#999AAA >注意，一定要拉取 1.4.1 的版本，因为下面我给的配置文件是 1.4.1 的，其他版本都会启动失败，Nacos 就是这么的恶心。拉取有点慢，耐心等待。

```bash
docker pull nacos/nacos-server:1.4.1
```

##  启动Nacos容器

<font color=#999AAA >全都粘贴到命令行一直回车就行了，不是一行一行粘的哈


```bash
docker run -d --name nacos \
-e PREFER_HOST_MODE=hostname \
-e MODE=standalone \
-e SPRING_DATASOURCE_PLATFORM=mysql \
-e MYSQL_SERVICE_HOST=111.111.111.111 \
-e MYSQL_SERVICE_PORT=3306 \
-e MYSQL_SERVICE_DB_NAME=nacos \
-e MYSQL_SERVICE_USER=root \
-e MYSQL_SERVICE_PASSWORD=root \
-p 8848:8848 nacos/nacos-server:1.4.1
```

<font color=#999AAA >上面的 111.111.111.111 写你的 MySQL 数据库的 IP，然后是端口，数据库名称 ==（数据库名称一定要是 nacos，这是官方指定的，nacos库要自己创建，sql 文件官网下载的压缩包解压出来就有，为了方便我下面教你们安装 MySQL 和建库的时候直接贴给你们）== ，账号，密码


##  开放防火墙端口

### CentOS

```bash
systemctl status firewalld    #首先查看火墙状态，如果防火墙没开，下面的不用看了
```

```bash
firewall-cmd --zone=public --add-port=8848/tcp --permanent #意思是放行 8848 端口
```
```bash
firewall-cmd --zone=public --add-port=3306/tcp --permanent #你指定的数据库的端口也要放行
```

```bash
firewall-cmd --reload   ##重新加载火墙,使放行的端口生效
```

### Debian/Ubuntu
分为两种，有些服务器安装的是 iptables 防火墙，有些是 ufw

```bash
iptables -I INPUT -p tcp --dport 8848 -j ACCEPT
iptables -I INPUT -p tcp --dport 3306 -j ACCEPT
netfilter-persistent save
```

```bash
sudo ufw allow 8848
sudo ufw allow 3306
sudo ufw reload
```


## 在 MySQL 中创建 nacos 数据库

<font color=#999AAA > nacos 数据库用于给 nacos 指定数据源，用于持久化配置文件到硬盘，Docker 安装 MySQL 的方法后面这里就不赘述了


<font color=#999AAA >进入 MySQL 容器，创建 nacos 数据库，切换到创建的 nacos 数据库，基本操作代码自己敲
![在这里插入图片描述](https://img-blog.csdnimg.cn/54f4dc6d37a4405eaec7530abc9e1f3d.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA57mB5Y2O5bC95aS05ruh5piv5q6H,size_13,color_FFFFFF,t_70,g_se,x_16)

<font color=#999AAA >==然后！！！把下面的 SQL 文件全都复制到 mysql> 后面，然后等它运行完，运行完以后看看是不是最后一句话没有运行，我的最后一句就没有运行，再敲一下回车就 OK==
![在这里插入图片描述](https://img-blog.csdnimg.cn/1e5aaa4d623944d18d45db2d9ca753fe.png)


```sql
CREATE TABLE `config_info` (

  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT 'id',

  `data_id` varchar(255) NOT NULL COMMENT 'data_id',

  `group_id` varchar(255) DEFAULT NULL,

  `content` longtext NOT NULL COMMENT 'content',

  `md5` varchar(32) DEFAULT NULL COMMENT 'md5',

  `gmt_create` datetime NOT NULL DEFAULT '2010-05-05 00:00:00' COMMENT '创建时间',

  `gmt_modified` datetime NOT NULL DEFAULT '2010-05-05 00:00:00' COMMENT '修改时间',

  `src_user` text COMMENT 'source user',

  `src_ip` varchar(20) DEFAULT NULL COMMENT 'source ip',

  `app_name` varchar(128) DEFAULT NULL,

  `tenant_id` varchar(128) DEFAULT '' COMMENT '租户字段',

  `c_desc` varchar(256) DEFAULT NULL,

  `c_use` varchar(64) DEFAULT NULL,

  `effect` varchar(64) DEFAULT NULL,

  `type` varchar(64) DEFAULT NULL,

  `c_schema` text,

  PRIMARY KEY (`id`),

  UNIQUE KEY `uk_configinfo_datagrouptenant` (`data_id`,`group_id`,`tenant_id`)

) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='config_info';



CREATE TABLE `config_info_aggr` (

  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT 'id',

  `data_id` varchar(255) NOT NULL COMMENT 'data_id',

  `group_id` varchar(255) NOT NULL COMMENT 'group_id',

  `datum_id` varchar(255) NOT NULL COMMENT 'datum_id',

  `content` longtext NOT NULL COMMENT '内容',

  `gmt_modified` datetime NOT NULL COMMENT '修改时间',

  `app_name` varchar(128) DEFAULT NULL,

  `tenant_id` varchar(128) DEFAULT '' COMMENT '租户字段',

  PRIMARY KEY (`id`),

  UNIQUE KEY `uk_configinfoaggr_datagrouptenantdatum` (`data_id`,`group_id`,`tenant_id`,`datum_id`)

) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='增加租户字段';



CREATE TABLE `config_info_beta` (

  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT 'id',

  `data_id` varchar(255) NOT NULL COMMENT 'data_id',

  `group_id` varchar(128) NOT NULL COMMENT 'group_id',

  `app_name` varchar(128) DEFAULT NULL COMMENT 'app_name',

  `content` longtext NOT NULL COMMENT 'content',

  `beta_ips` varchar(1024) DEFAULT NULL COMMENT 'betaIps',

  `md5` varchar(32) DEFAULT NULL COMMENT 'md5',

  `gmt_create` datetime NOT NULL DEFAULT '2010-05-05 00:00:00' COMMENT '创建时间',

  `gmt_modified` datetime NOT NULL DEFAULT '2010-05-05 00:00:00' COMMENT '修改时间',

  `src_user` text COMMENT 'source user',

  `src_ip` varchar(20) DEFAULT NULL COMMENT 'source ip',

  `tenant_id` varchar(128) DEFAULT '' COMMENT '租户字段',

  PRIMARY KEY (`id`),

  UNIQUE KEY `uk_configinfobeta_datagrouptenant` (`data_id`,`group_id`,`tenant_id`)

) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='config_info_beta';


CREATE TABLE `config_info_tag` (

  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT 'id',

  `data_id` varchar(255) NOT NULL COMMENT 'data_id',

  `group_id` varchar(128) NOT NULL COMMENT 'group_id',

  `tenant_id` varchar(128) DEFAULT '' COMMENT 'tenant_id',

  `tag_id` varchar(128) NOT NULL COMMENT 'tag_id',

  `app_name` varchar(128) DEFAULT NULL COMMENT 'app_name',

  `content` longtext NOT NULL COMMENT 'content',

  `md5` varchar(32) DEFAULT NULL COMMENT 'md5',

  `gmt_create` datetime NOT NULL DEFAULT '2010-05-05 00:00:00' COMMENT '创建时间',

  `gmt_modified` datetime NOT NULL DEFAULT '2010-05-05 00:00:00' COMMENT '修改时间',

  `src_user` text COMMENT 'source user',

  `src_ip` varchar(20) DEFAULT NULL COMMENT 'source ip',

  PRIMARY KEY (`id`),

  UNIQUE KEY `uk_configinfotag_datagrouptenanttag` (`data_id`,`group_id`,`tenant_id`,`tag_id`)

) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='config_info_tag';



CREATE TABLE `config_tags_relation` (

  `id` bigint(20) NOT NULL COMMENT 'id',

  `tag_name` varchar(128) NOT NULL COMMENT 'tag_name',

  `tag_type` varchar(64) DEFAULT NULL COMMENT 'tag_type',

  `data_id` varchar(255) NOT NULL COMMENT 'data_id',

  `group_id` varchar(128) NOT NULL COMMENT 'group_id',

  `tenant_id` varchar(128) DEFAULT '' COMMENT 'tenant_id',

  `nid` bigint(20) NOT NULL AUTO_INCREMENT,

  PRIMARY KEY (`nid`),

  UNIQUE KEY `uk_configtagrelation_configidtag` (`id`,`tag_name`,`tag_type`),

  KEY `idx_tenant_id` (`tenant_id`)

) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='config_tag_relation';



CREATE TABLE `group_capacity` (

  `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT COMMENT '主键ID',

  `group_id` varchar(128) NOT NULL DEFAULT '' COMMENT 'Group ID，空字符表示整个集群',

  `quota` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '配额，0表示使用默认值',

  `usage` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '使用量',

  `max_size` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '单个配置大小上限，单位为字节，0表示使用默认值',

  `max_aggr_count` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '聚合子配置最大个数，，0表示使用默认值',

  `max_aggr_size` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '单个聚合数据的子配置大小上限，单位为字节，0表示使用默认值',

  `max_history_count` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '最大变更历史数量',

  `gmt_create` datetime NOT NULL DEFAULT '2010-05-05 00:00:00' COMMENT '创建时间',

  `gmt_modified` datetime NOT NULL DEFAULT '2010-05-05 00:00:00' COMMENT '修改时间',

  PRIMARY KEY (`id`),

  UNIQUE KEY `uk_group_id` (`group_id`)

) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='集群、各Group容量信息表';



CREATE TABLE `his_config_info` (

  `id` bigint(64) unsigned NOT NULL,

  `nid` bigint(20) unsigned NOT NULL AUTO_INCREMENT,

  `data_id` varchar(255) NOT NULL,

  `group_id` varchar(128) NOT NULL,

  `app_name` varchar(128) DEFAULT NULL COMMENT 'app_name',

  `content` longtext NOT NULL,

  `md5` varchar(32) DEFAULT NULL,

  `gmt_create` datetime NOT NULL DEFAULT '2010-05-05 00:00:00',

  `gmt_modified` datetime NOT NULL DEFAULT '2010-05-05 00:00:00',

  `src_user` text,

  `src_ip` varchar(20) DEFAULT NULL,

  `op_type` char(10) DEFAULT NULL,

  `tenant_id` varchar(128) DEFAULT '' COMMENT '租户字段',

  PRIMARY KEY (`nid`),

  KEY `idx_gmt_create` (`gmt_create`),

  KEY `idx_gmt_modified` (`gmt_modified`),

  KEY `idx_did` (`data_id`)

) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='多租户改造';



CREATE TABLE `tenant_capacity` (

  `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT COMMENT '主键ID',

  `tenant_id` varchar(128) NOT NULL DEFAULT '' COMMENT 'Tenant ID',

  `quota` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '配额，0表示使用默认值',

  `usage` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '使用量',

  `max_size` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '单个配置大小上限，单位为字节，0表示使用默认值',

  `max_aggr_count` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '聚合子配置最大个数',

  `max_aggr_size` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '单个聚合数据的子配置大小上限，单位为字节，0表示使用默认值',

  `max_history_count` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '最大变更历史数量',

  `gmt_create` datetime NOT NULL DEFAULT '2010-05-05 00:00:00' COMMENT '创建时间',

  `gmt_modified` datetime NOT NULL DEFAULT '2010-05-05 00:00:00' COMMENT '修改时间',

  PRIMARY KEY (`id`),

  UNIQUE KEY `uk_tenant_id` (`tenant_id`)

) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='租户容量信息表';





CREATE TABLE `tenant_info` (

  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT 'id',

  `kp` varchar(128) NOT NULL COMMENT 'kp',

  `tenant_id` varchar(128) default '' COMMENT 'tenant_id',

  `tenant_name` varchar(128) default '' COMMENT 'tenant_name',

  `tenant_desc` varchar(256) DEFAULT NULL COMMENT 'tenant_desc',

  `create_source` varchar(32) DEFAULT NULL COMMENT 'create_source',

  `gmt_create` bigint(20) NOT NULL COMMENT '创建时间',

  `gmt_modified` bigint(20) NOT NULL COMMENT '修改时间',

  PRIMARY KEY (`id`),

  UNIQUE KEY `uk_tenant_info_kptenantid` (`kp`,`tenant_id`),

  KEY `idx_tenant_id` (`tenant_id`)

) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='tenant_info';



CREATE TABLE users (

	username varchar(50) NOT NULL PRIMARY KEY,

	password varchar(500) NOT NULL,

	enabled boolean NOT NULL

);



CREATE TABLE roles (

	username varchar(50) NOT NULL,

	role varchar(50) NOT NULL,

	constraint uk_username_role UNIQUE (username,role)

);



CREATE TABLE permissions (

    role varchar(50) NOT NULL,

    resource varchar(512) NOT NULL,

    action varchar(8) NOT NULL,

    constraint uk_role_permission UNIQUE (role,resource,action)

);



INSERT INTO users (username, password, enabled) VALUES ('nacos', '$2a$10$EuWPZHzz32dJN7jexM34MOeYirDdFAZm2kuWj7VEOJhhZkDrxfvUu', TRUE);



INSERT INTO roles (username, role) VALUES ('nacos', 'ROLE_ADMIN');


```


<font color=#999AAA >==最重要的是！！！，要记得重启一下 nacos==

```bash
docker restart nacos
```


##  登录 nacos
<font color=#999AAA >如果你的 Linux 服务器没有界面，就在一台 Windows 电脑上登录，因为上面防火墙已经开放端口，浏览器地址栏输入 IP:8848/nacos 就可以了。默认账号密码都是 nacos
![在这里插入图片描述](https://img-blog.csdnimg.cn/ce58b52eb55a4b9da6aba8c50d0c2a81.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA57mB5Y2O5bC95aS05ruh5piv5q6H,size_20,color_FFFFFF,t_70,g_se,x_16)
<font color=#999AAA >然后按照我下面的标记，创建新的 命名空间 ，默认的命名空间是 public，不建议用，最好一个项目一个命名空间。

![在这里插入图片描述](https://img-blog.csdnimg.cn/debf3f5f121b4b488aad297ad712aeb0.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA57mB5Y2O5bC95aS05ruh5piv5q6H,size_20,color_FFFFFF,t_70,g_se,x_16)

<font color=#999AAA >回到我们的配置列表，点击刚刚创建的命名空间

![在这里插入图片描述](https://img-blog.csdnimg.cn/077b8535812f4c4cb5f0ff4f46977dcf.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA57mB5Y2O5bC95aS05ruh5piv5q6H,size_20,color_FFFFFF,t_70,g_se,x_16)
<font color=#999AAA >点击右边蓝色 + 号，创建一个 yaml，这个 Data Id 建议都是这个格式，如果项目有多个配置文件，就 application-xx.yaml ，默认就起这个名字 application.yaml

<font color=#999AAA >Group 的话默认是这个，建议改一下，不改的话可能会出问题，只是可能，我就先不改了，然后下面类型选择 yaml，把你的 springboot 项目配置文件全都粘贴到下面的框里。

![在这里插入图片描述](https://img-blog.csdnimg.cn/391e697f19e44a858a32635399ca6ed2.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA57mB5Y2O5bC95aS05ruh5piv5q6H,size_20,color_FFFFFF,t_70,g_se,x_16)

<font color=#999AAA >Nacos 的安装和配置就完成了，下面我们开始把 Springboot 的配置文件放到 nacos 并使用


<hr style=" border:solid; width:100px; height:1px;" color=#000000 size=1">


# 二、把 Springboot 的配置文件放到 nacos 并使用
## 1.引入依赖

<font color=#999AAA >只需要这两个，网上那些引了一堆的不要信，版本不要改，nacos 很恶心

==Maven这样引==

```xml
<dependency>
    <groupId>com.alibaba.nacos</groupId>
    <artifactId>nacos-client</artifactId>
    <version>1.4.1</version>
</dependency>
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-stream-rocketmq</artifactId>
    <version>2.2.0.RELEASE</version>
</dependency>
```

==Gradle这样引==
```xml
compile group: 'com.alibaba.cloud', name: 'spring-cloud-stream-binder-rocketmq', version: '2.2.0.RELEASE'
compile group: 'com.alibaba.nacos', name: 'nacos-client', version: '1.4.1'
```




## 2.配置 nacos 的yaml

<font color=#999AAA >在 resources 下面新建 bootstrap.properties ，yml 也行
![在这里插入图片描述](https://img-blog.csdnimg.cn/96b61afa70a14b4cb94d094346d2131e.png)

<font color=#999AAA >里面这样写

```java
spring.application.name=test #不重要，可以不写，写你的项目名就行
spring.cloud.nacos.config.group=DEFAULT_GROUP #这个组就是上面创建的时候填的那个
spring.main.allow-bean-definition-overriding=true

#nacos配置中心
spring.cloud.nacos.config.server-addr=111.111.111.111:8848  #这个IP就是你的nacos的地址
#命名空间
spring.cloud.nacos.config.namespace=51f82ad7-11fd-413a-9b0a-fd11ee1213d7 #这个是你创建的命名空间 test 右边的那一串很长的字符，看我上面的图片
#默认加载配置前缀后缀
spring.cloud.nacos.config.prefix=application
spring.cloud.nacos.config.file-extension=yaml

#额外的配置文件，这个就是我上面说的如果一个项目有多个配置文件要启动，就可以在nacos再新建一个文件，我只用一个，所以下面的注释掉
#database
#spring.cloud.nacos.config.shared-configs[0].data-id=application-database.yaml
#spring.cloud.nacos.config.shared-configs[0].group=DEFAULT_GROUP
#spring.cloud.nacos.config.shared-configs[0].refresh=true

#这一句true就是优先使用nacos里的配置文件，注意只是优先，也就是说，nacos和本地都有的配置，nacos的起作用，如果本地有的而nacos没有的配置，本地的那个比nacos多出来的配置还是会加载的
#如果是false就是不使用nacos里的，只是本地配置文件生效
spring.cloud.nacos.config.enabled=true
```

<font color=#999AAA >然后全部结束，直接启动项目吧


<hr style=" border:solid; width:100px; height:1px;" color=#000000 size=1">

# 总结
<font color=#999AAA >本篇的 nacos 是单机部署，后面研究研究配合 Dubbo 的集群部署吧
