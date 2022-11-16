---
title: Springboot配置Druid或Hikari连接池（并开启MyBatisPlus事务）
date: 2021-12-20 22:35:00
categories: Java
tags:
    - Druid
    - Hikari
    - 事务
    - MyBatisPlus
cover: /img/springbootLogo.jpeg
---
# 前言

<font color=#999AAA >阿里巴巴的 `Druid` 连接池应该是咱国内用的最广泛的连接池了，而Hikari是Springboot2以后默认最优先使用的连接池。</font>

<font color=#999AAA > `Hikari` 连接池的特点就是极快，比 Druid 快了不止一个数量级，而功能较少，Druid的功能很全面，防火墙、拦截器、监控、检测慢SQL等</font>

<font color=#999AAA >选哪一个根据需求，我个人做项目不需要太多的功能，快就完事，所以我用 Hikari</font>

<hr style=" border:solid; width:100px; height:1px;" color=#000000 size=1">


# 一、引入和配置 Hikari
##  依赖



<font color=#999AAA >依赖里面，引入 spring-boot-starter-jdbc 就可以了，包含Hikar。

<font color=#999AAA >给你们讲一下哦，Mybatis-Plus 一定要引 `mybatis-plus-boot-starter`，不要引 `mybatisplus`，因为 `mybatis-plus-boot-starter`专门为Springboot打造，引入这个依赖可以不用配置任何 xml 文件，Springboot 和 Spring 最大的区别就是不需要 xml 配置，网上那些配置 xml 的都是用的旧依赖，没有跟上时代的潮流。

```xml
		<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.47</version>
        </dependency>
        <!--此依赖默认使用Hikari连接池-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jdbc</artifactId>
        </dependency>
        <!-- MybatisPlus 核心库 -->
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-boot-starter</artifactId>
            <version>3.4.2</version>
        </dependency>
		
```


##  配置yml

```yaml
server:
  port: 808
spring:
  datasource:
    driver-class-name: com.mysql.jdbc.Driver
    url: jdbc:mysql://IP:3306/xuyijie_icu?useSSL=false&useUnicode=true&characterEncoding=UTF-8
    username:
    password:
    type: com.zaxxer.hikari.HikariDataSource
    hikari:
    # 连接池中允许的最小连接数。缺省值：10
      minimum-idle: 10
    # 连接池中允许的最大连接数。缺省值：10
      maximum-pool-size: 100
    # 自动提交
      auto-commit: true
    # 一个连接idle状态的最大时长（毫秒），超时则被释放（retired），缺省:10分钟
      idle-timeout: 600
    # 连接池名字
      pool-name: 泡泡的HikariCP
    # 一 个连接的生命时长（毫秒），超时而且没被使用则被释放（retired），缺省:30分钟，建议设置比数据库超时时长少30秒
      max-lifetime: 1800000
    # 等待连接池分配连接的最大时长（毫秒），超过这个时长还没可用的连接则发生SQLException， 缺省:30秒
      connection-timeout: 30000    
mybatis-plus:
  mapper-locations: classpath*:mapper/*.xml
  # 打印出SQL语句
  configuration:
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
```

## 将事务交由Springboot管理

<font color=#999AAA >如果没有这一步的话，查询时连接池会报 synchronization 的错，意思是连接池的事务没有开启给Springboot管理，而且每次连接都要创建新的SqlSession，失去了连接池的意义。（`具体什么是事务，大概解释一下，假如你的程序正在进行数据库写操作，但是中途程序报错了，所以数据库里被写入了一些残缺的东西，如果开启了事务功能，springboot会在程序正常结束后才会写入数据库，如果程序中途报错，事务会回滚，也就是数据库不会被残缺的数据污染`，==一句话，不开启事务是实时写入，写入语句执行完数据库就有数据了，开启事务，是所有程序执行完，数据库才有数据。==）

```bash
Closing non transactional SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@6d8811ab]
Creating a new SqlSession
SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@4dd82b9a] was not registered for synchronization because synchronization is not active
```

<font color=#999AAA >配置方法太简单了，就是在启动类上添加一个注解 `@EnableTransactionManagement`，最好自己手打然后回车键自动 import class，复制粘贴的话可能无法找到这个注解。

![在这里插入图片描述](https://img-blog.csdnimg.cn/3083c337c466404b8fb638559d5a567b.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA57mB5Y2O5bC95aS05ruh5piv5q6H,size_20,color_FFFFFF,t_70,g_se,x_16)

<font color=#999AAA >然后在每个 Service 上都添加一个注解 `@Transactional`
![在这里插入图片描述](https://img-blog.csdnimg.cn/2ad1344ad981465387f1de32fe2cd31b.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA57mB5Y2O5bC95aS05ruh5piv5q6H,size_19,color_FFFFFF,t_70,g_se,x_16)
<font color=#999AAA >然后查询的时候没有报错出现以下信息，就是配置成功了

```bash
Registering transaction synchronization for SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@3c34c1ee]
JDBC Connection [HikariProxyConnection@405147465 wrapping com.mysql.jdbc.JDBC4Connection@7e6e118a] will be managed by Spring
Releasing transactional SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@3c34c1ee]
Fetched SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@3c34c1ee] from current transactio
```


<hr style=" border:solid; width:100px; height:1px;" color=#000000 size=1">


# 二、引入和配置 Druid
## 依赖

<font color=#999AAA >给你们讲一下哦，一定要引 `druid-spring-boot-starter`，不要引 `druid`，原因和上面 Mybatis-plus 的依赖一样，这样就不用配置 xml 注入数据源。

```xml
		<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.47</version>
        </dependency>
        <!-- mybatisPlus 核心库 -->
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-boot-starter</artifactId>
            <version>3.4.2</version>
        </dependency>
        <!-- 引入阿里数据库连接池，暂时注掉，听说springboot2默认使用的Hikari连接池最快 -->
        <dependency>-->
           <groupId>com.alibaba</groupId>-->
            <artifactId>druid-spring-boot-starter</artifactId>-->
            <version>1.2.8</version>-->
        </dependency>-->
		
```


## 配置yml


```yaml
server:
  port: 8081
spring:
  datasource:
    driver-class-name: com.mysql.jdbc.Driver
    url: jdbc:mysql://IP:3306/xuyijie_icu?useSSL=false&useUnicode=true&characterEncoding=UTF-8
    username:
    password:
    type: com.alibaba.druid.pool.DruidDataSource
    druid:
      initial-size: 10 # 初始化时建立物理连接的个数。初始化发生在显示调用init方法，或者第一次getConnection时
      min-idle: 10 # 最小连接池数量
      maxActive: 200 # 最大连接池数量
      maxWait: 60000 # 获取连接时最大等待时间，单位毫秒。配置了maxWait之后，缺省启用公平锁，并发效率会有所下降，如果需要可以通过配置
      timeBetweenEvictionRunsMillis: 60000 # 关闭空闲连接的检测时间间隔.Destroy线程会检测连接的间隔时间，如果连接空闲时间大于等于minEvictableIdleTimeMillis则关闭物理连接。
      minEvictableIdleTimeMillis: 300000 # 连接的最小生存时间.连接保持空闲而不被驱逐的最小时间
      validationQuery: SELECT 1 FROM DUAL # 验证数据库服务可用性的sql.用来检测连接是否有效的sql 因数据库方言而差, 例如 oracle 应该写成 SELECT 1 FROM DUAL
      testWhileIdle: true # 申请连接时检测空闲时间，根据空闲时间再检测连接是否有效.建议配置为true，不影响性能，并且保证安全性。申请连接的时候检测，如果空闲时间大于timeBetweenEvictionRun
      testOnBorrow: false # 申请连接时直接检测连接是否有效.申请连接时执行validationQuery检测连接是否有效，做了这个配置会降低性能。
      testOnReturn: false # 归还连接时检测连接是否有效.归还连接时执行validationQuery检测连接是否有效，做了这个配置会降低性能。
      poolPreparedStatements: true # 开启PSCache
      maxPoolPreparedStatementPerConnectionSize: 20 #设置PSCache值
      connectionErrorRetryAttempts: 3 # 连接出错后再尝试连接三次
      breakAfterAcquireFailure: true # 数据库服务宕机自动重连机制
      timeBetweenConnectErrorMillis: 300000 # 连接出错后重试时间间隔
      asyncInit: true # 异步初始化策略
      remove-abandoned: true # 是否自动回收超时连接
      remove-abandoned-timeout: 1800 # 超时时间(以秒数为单位)
      transaction-query-timeout: 6000 # 事务超时时间
      filters: stat,wall,log4j2
      useGlobalDataSourceStat: true #合并多个DruidDataSource的监控数据
      connectionProperties: druid.stat.mergeSql\=true;druid.stat.slowSqlMillis\=5000 #通过connectProperties属性来打开mergeSql功能；慢SQL记录
mybatis-plus:
  mapper-locations: classpath*:mapper/*.xml
  # 打印出SQL语句
  configuration:
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
```


## 将事务交由Springboot管理（和Hikari一样）

<hr style=" border:solid; width:100px; height:1px;" color=#000000 size=1">

# 总结
<font color=#999AAA >这是一个总结
