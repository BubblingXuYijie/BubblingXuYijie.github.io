---
title: Springboot引入hibernate配置自动建表并进行增删改查
date: 2022-09-19 18:23:15
categories: Linux
tags:
    - SpringBoot
    - hibernate
cover: https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/hibernateLogo.jpg
---
# 前言
> 有些业务比较复杂，比如我们需要新建10张表，每张表有10个字段，如果用手工来操作，肯定非常浪费时间，而且随着代码中对实体类的修改，还要同时修改数据库表，有时候写着写着就忘了，代码改了，数据库没改，这种问题使用 hibernate 的自动建表就好啦。

---


# 一、引入依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <scope>runtime</scope>
</dependency>
```



# 二、配置yml
>自动建表的配置是ddl-auto，有多个属性可选
# 1. none 永远以数据表字段为准，不做任何修改
# 2. validate 加载hibernate时，验证创建数据库表结构，会和数据库中的表进行比较，不会创建新表，但是会插入新值
# 3. create 每次加载hibernate，重新创建数据库表结构，这就是导致数据库表数据丢失的原因
# 4. create-drop 加载hibernate时创建，退出是删除表结构
# 5. update 加载hibernate自动更新数据库结构（最常用的一个）
```yaml
server:
  port: 8081
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://127.0.0.1:3306/test?useSSL=false&useUnicode=true&characterEncoding=UTF-8&serverTimezone=Asia/Shanghai&rewriteBatchedStatements=true
    username: root
    password: root
  jpa:
    hibernate:
      ddl-auto: update
    # 打印SQL语句
    show-sql: true
```

# 三、写代码
## 1、新建数据库（空数据库即可，不要新建表）
## 2、实体类


>@Id代表这是主键，@GeneratedValue和@GenericGenerator设置主键策略是UUID
>@Column可以不写，name是数据库中的字段名，如果数据库中要新建的对应字段也叫name，可以不写，columnDefinition指定字段在数据库中的类型、长度、注释等
```java
package com.xuyijie.test.entity;

import jakarta.persistence.*;
import lombok.Data;
import org.hibernate.annotations.GenericGenerator;

/**
 * @author 徐一杰
 * @date 2022/9/19 17:25
 * @description
 */
//JPA(Hibernate)的实体类注解
@Entity
//表名
@Table(name = "people")
//Lombok
@Data
public class People {
    
    //Id代表这是主键，GeneratedValue和GenericGenerator设置主键策略是UUID
    @Id
    @GeneratedValue(generator = "id")
    @GenericGenerator(name = "id", strategy = "uuid.hex")
    private String id;

    //Column可以不写，name是数据库中的字段名，如果数据库中要新建的对应字段也叫name，可以不写，columnDefinition指定字段在数据库中的类型、长度、注释等
    @Column(name = "name", columnDefinition="varchar(255) NOT NULL COMMENT '名字'")
    private String name;

    @Column(name = "sex", columnDefinition="varchar(2) NOT NULL COMMENT '性别'")
    private String sex;

    @Column(name = "age", columnDefinition="int")
    private Integer age;
}

```

## 3、Dao层
>JpaRepository<People, String>尖括号里面填写的是实体类和实体类的主键数据类型，PeopleMapper继承JpaRepository以后，可以把PeopleMapper 注入到Service里，使用很多内置方法

```java
package com.xuyijie.test.mapper;

import com.xuyijie.test.entity.People;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

/**
 * @author 徐一杰
 * @date 2022/9/19 17:42
 * @description
 */
@Repository
public interface PeopleMapper extends JpaRepository<People, String> {
}

```

## 4、Controller

```java
package com.xuyijie.test.controller;

import com.xuyijie.test.entity.People;
import com.xuyijie.test.mapper.PeopleMapper;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

/**
 * @author 徐一杰
 * @date 2022/9/19 17:12
 * @description
 */
@RestController
@RequestMapping("/test")
public class Test {
    @Autowired
    private PeopleMapper peopleMapper;

    @GetMapping("/hello/{str}")
    public String Hello(@PathVariable String str){
        People people = new People();
        people.setName(str);
        people.setSex("男");
        // 这里的save和findAll都是hibernate自带的方法，里面还有很多内置方法
        peopleMapper.save(people);
        System.out.println(peopleMapper.findAll());
        return str;
    }
}

```

# 四、测试结果
## 1、表已建好
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/hibernate0.png)
## 2、请求接口，打印SQL，插入和查询数据
使用浏览器调用接口

![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/hibernate1.png)
控制台打印出SQL语句，查询出我们刚刚插入的一条数据，主键为自动生成的UUID

![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/hibernate2.png)
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/hibernate3.png)



# 总结

