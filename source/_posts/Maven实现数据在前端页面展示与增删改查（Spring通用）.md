---
title: Maven实现数据在前端页面展示与增删改查（Spring通用）
date: 2020-07-02 14:52:24
categories: Java
tags:
    - Maven
    - Spring
cover: https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/Maven0.png
---
# 项目开始，先上源码下载，点击下载

不需要积分，直接下载

[Mave前后端的数据展示与操作源码](https://download.csdn.net/download/qq_48922459/52839676)

<hr>


#  数据库准备
## 安装MySQL
[MySQL安装地址](https://dev.mysql.com/downloads/windows/installer/8.0.html)
官网打开有点慢的，选择这个下载安装，傻瓜式一键到底
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/Maven1.png)



## 建立数据库

使用Navcat或SQLyog建立创建一个数据库：wzsxy，在这个数据库上创建一张表：tb_user，并输入几条起始数据
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/Maven2.png)
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/Maven3.png)
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/Maven4.png)

## 使用idea快速生成一个Maven工程
选择新建项目，左边的“Maven”，勾选“Create from archetype”，选择下方目录的maven-archetype-webapp，然后next
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/Maven5.png)
到了这一步，如果没有安装和配置过Maven，就选用idea自带的“Bundled(Maven 3)”，你们的和我可能不太一样，下面的两行是我自己配置的Maven设置和仓库位置，你们的选择“Bundled(Maven 3)”其他保持默认即可
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/Maven6.png)
初始的项目目录
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/Maven7.png)
我们最终搭建好的项目目录，大家可以先把目录建立好
解释一下bean包、dao包
1、bean 实体层 实体类 --> 属性，构造，方法
2、controller层： 控制层 控制业务逻辑
3、dao 持久层 --> 数据库的操作
4、service层：业务层 控制业务
fliter则是对未登录的网页进行拦截的操作
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/Maven8.png)
webapp下面就是我们的网页，也就是要展示和操作数据的地方，我已经配置好了网页(包含SQL代码)，源码资源里面有，在文章开头，可以无需积分下载。



## 配置pom.xml文件
在pom中导入项目需要的dependencies依赖，依赖在下方代码中

```java
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>org.example</groupId>
  <artifactId>maven</artifactId>
  <version>1.0-SNAPSHOT</version>
  <packaging>war</packaging>

  <name>maven Maven Webapp</name>
  <!-- FIXME change it to the project's website -->
  <url>http://www.example.com</url>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <maven.compiler.source>1.8</maven.compiler.source>
    <maven.compiler.target>1.8</maven.compiler.target>
    <!-- spring版本号 -->
    <spring.version>5.0.2.RELEASE</spring.version>
    <!-- mybatis版本号 -->
    <mybatis.version>3.2.6</mybatis.version>
    <!-- log4j日志文件管理包版本 -->
    <slf4j.version>1.7.7</slf4j.version>
    <log4j.version>1.2.17</log4j.version>
    <c3p0.version>0.9.5.2</c3p0.version>
    <taglibs.version>1.1.2</taglibs.version>
  </properties>

  <dependencies>
    <!-- spring核心包 -->
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-core</artifactId>
      <version>${spring.version}</version>
    </dependency>

    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-web</artifactId>
      <version>${spring.version}</version>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-oxm</artifactId>
      <version>${spring.version}</version>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-tx</artifactId>
      <version>${spring.version}</version>
    </dependency>

    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-jdbc</artifactId>
      <version>${spring.version}</version>
    </dependency>

    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-webmvc</artifactId>
      <version>${spring.version}</version>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-aop</artifactId>
      <version>${spring.version}</version>
    </dependency>

    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-context-support</artifactId>
      <version>${spring.version}</version>
    </dependency>

    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-test</artifactId>
      <version>${spring.version}</version>
    </dependency>
    <!-- mybatis核心包 -->
    <dependency>
      <groupId>org.mybatis</groupId>
      <artifactId>mybatis</artifactId>
      <version>${mybatis.version}</version>
    </dependency>
    <!-- mybatis/spring包 -->
    <dependency>
      <groupId>org.mybatis</groupId>
      <artifactId>mybatis-spring</artifactId>
      <version>1.2.2</version>
    </dependency>
    <!-- 导入java ee jar 包 -->
    <dependency>
      <groupId>javax</groupId>
      <artifactId>javaee-api</artifactId>
      <version>7.0</version>
    </dependency>

    <!-- 导入Mysql数据库链接jar包 -->
    <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
      <version>5.1.30</version>
    </dependency>
    <!-- 导入dbcp的jar包，用来在applicationContext.xml中配置数据库 -->
    <dependency>
      <groupId>commons-dbcp</groupId>
      <artifactId>commons-dbcp</artifactId>
      <version>1.2.2</version>
    </dependency>
    <!-- JSTL标签类 -->
    <dependency>
      <groupId>jstl</groupId>
      <artifactId>jstl</artifactId>
      <version>1.2</version>
    </dependency>
    <!-- 日志文件管理包 -->
    <!-- log start -->
    <dependency>
      <groupId>log4j</groupId>
      <artifactId>log4j</artifactId>
      <version>${log4j.version}</version>
    </dependency>


    <!-- 数据连接池 -->
    <dependency>
      <groupId>com.mchange</groupId>
      <artifactId>c3p0</artifactId>
      <version>${c3p0.version}</version>
    </dependency>

    <dependency>
      <groupId>taglibs</groupId>
      <artifactId>standard</artifactId>
      <version>${taglibs.version}</version>
    </dependency>

    <dependency>
      <groupId>org.slf4j</groupId>
      <artifactId>slf4j-api</artifactId>
      <version>${slf4j.version}</version>
    </dependency>
    <dependency>
      <groupId>org.slf4j</groupId>
      <artifactId>slf4j-log4j12</artifactId>
      <version>${slf4j.version}</version>
    </dependency>

    <!-- 导入servlet-api/jsp -->
    <dependency>
      <groupId>javax.servlet</groupId>
      <artifactId>javax.servlet-api</artifactId>
      <version>3.1.0</version>
      <scope>provided</scope>
    </dependency>
    <!-- https://mvnrepository.com/artifact/javax.servlet.jsp/javax.servlet.jsp-api -->
    <dependency>
      <groupId>javax.servlet.jsp</groupId>
      <artifactId>javax.servlet.jsp-api</artifactId>
      <version>2.3.1</version>
      <scope>provided</scope>
    </dependency>
    <dependency>
      <groupId>org.mortbay.jetty</groupId>
      <artifactId>servlet-api</artifactId>
      <version>9.0.16</version>
    </dependency>

  </dependencies>

  <build>
    <finalName>maven</finalName>
    <pluginManagement><!-- lock down plugins versions to avoid using Maven defaults (may be moved to parent pom) -->
      <plugins>
        <plugin>
          <artifactId>maven-clean-plugin</artifactId>
          <version>3.1.0</version>
        </plugin>
        <!-- see http://maven.apache.org/ref/current/maven-core/default-bindings.html#Plugin_bindings_for_war_packaging -->
        <plugin>
          <artifactId>maven-resources-plugin</artifactId>
          <version>3.0.2</version>
        </plugin>
        <plugin>
          <artifactId>maven-compiler-plugin</artifactId>
          <version>3.8.0</version>
        </plugin>
        <plugin>
          <artifactId>maven-surefire-plugin</artifactId>
          <version>2.22.1</version>
        </plugin>
        <plugin>
          <artifactId>maven-war-plugin</artifactId>
          <version>3.2.2</version>
        </plugin>
        <plugin>
          <artifactId>maven-install-plugin</artifactId>
          <version>2.5.2</version>
        </plugin>
        <plugin>
          <artifactId>maven-deploy-plugin</artifactId>
          <version>2.8.2</version>
        </plugin>
      </plugins>
    </pluginManagement>
  </build>
</project>

```

粘贴到pom文件中，然后点击pom页面右上方的刷新图标，等待所需jar包的下载，需要数分钟
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/Maven9.png)

## 配置resouces
### 配置db.properties
配置db.properties文件是与我们创建的数据库建立连接，username和password是你创建数据库的初始密码，如果修改过请填写为你修改过的密码


```java
jdbc.driver=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/wzsxy?useSSL=true&characterEncoding=utf-8
jdbc.username=root
jdbc.password=123456
```
### 配置applicationContext.xml

```java
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
<mapper namespace="com.XuYijie.dao.UserDao" >
    <select id="findUserByUserName" parameterType="String" resultType="com.XuYijie.bean.User">
        select * from tb_user where username=#{username}
    </select>

    <select id="findAll" resultType="com.XuYijie.bean.User" >
        select * from tb_user
        <if test="username!=null and username!=''">
             where username like concat("%",#{username},"%")
         </if>
         limit #{start},5
    </select>

    <delete id="deleteById" parameterType="int">
        delete from tb_user where id=#{id}
    </delete>

    <insert id="add" parameterType="user">
        insert into tb_user (username,password) values (#{username},#{password})
    </insert>

    <select id="selectById" parameterType="int" resultType="com.XuYijie.bean.User">
        select * from tb_user where id=#{id}
    </select>

    <update id="update" parameterType="user">
        update tb_user set username=#{username},password=#{password} where id=#{id}
    </update>

    <select id="getTotalCount" resultType="int">
        select count(*) from tb_user
        <if test="username!=null and username!=''">
            where username like concat("%",#{username},"%")
        </if>
    </select>
    
    <delete id="deleteAll" parameterType="list">
        delete from tb_user where id in 
        <foreach collection="ids" item="id" open="(" close=")" separator=",">
            #{id}
        </foreach>
    </delete>

</mapper>
```
## bean层
### PageInfo

```java
package com.XuYijie.bean;

import java.util.List;

public class PageInfo<T> {
    private List<T> list;
    private int totalPage;
    private int size;
    private int totalCount;
    private int currentPage;

    public List<T> getList() {
        return list;
    }

    public void setList(List<T> list) {
        this.list = list;
    }

    public int getTotalPage() {
        return totalPage;
    }

    public void setTotalPage(int totalPage) {
        this.totalPage = totalPage;
    }

    public int getSize() {
        return size;
    }

    public void setSize(int size) {
        this.size = size;
    }

    public int getTotalCount() {
        return totalCount;
    }

    public void setTotalCount(int totalCount) {
        this.totalCount = totalCount;
    }

    public int getCurrentPage() {
        return currentPage;
    }

    public void setCurrentPage(int currentPage) {
        this.currentPage = currentPage;
    }
}

```
## Role

```java
package com.XuYijie.bean;

public class Role {
    private int id;
    private String rolename;
    private String roledesc;

    @Override
    public String toString() {
        return "Role{" +
                "id=" + id +
                ", rolename='" + rolename + '\'' +
                ", roledesc='" + roledesc + '\'' +
                '}';
    }

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getRolename() {
        return rolename;
    }

    public void setRolename(String rolename) {
        this.rolename = rolename;
    }

    public String getRoledesc() {
        return roledesc;
    }

    public void setRoledesc(String roledesc) {
        this.roledesc = roledesc;
    }
}

```

## UserRole

```java
package com.XuYijie.bean;

public class UserRole {
    private int id;
    private int userId;
    private int roleId;

    @Override
    public String toString() {
        return "UserRole{" +
                "id=" + id +
                ", userId=" + userId +
                ", roleId=" + roleId +
                '}';
    }

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public int getUserId() {
        return userId;
    }

    public void setUserId(int userId) {
        this.userId = userId;
    }

    public int getRoleId() {
        return roleId;
    }

    public void setRoleId(int roleId) {
        this.roleId = roleId;
    }
}

```

### User

```java
package com.XuYijie.bean;

public class User {

    public User() {
    }

    public User(int id, String username, String password) {
        this.id = id;
        this.username = username;
        this.password = password;
    }

    public User(String username, String password) {
        this.username = username;
        this.password = password;
    }

    private int id;
    private String username;
    private String password;

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    @Override
    public String toString() {
        return "User{" +
                "id=" + id +
                ", username='" + username + '\'' +
                ", password='" + password + '\'' +
                '}';
    }
}

```
## controller层
### UserController

```java
package com.XuYijie.controller;

import com.XuYijie.bean.PageInfo;
import com.XuYijie.bean.Role;
import com.XuYijie.bean.User;
import com.XuYijie.service.IRoleService;
import com.XuYijie.service.IUserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.ResponseBody;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpSession;
import java.util.ArrayList;
import java.util.List;

@Controller
@RequestMapping("/user")
public class UserController {
    @Autowired
    private IUserService userService;
    @Autowired
    private IRoleService roleService;
    @RequestMapping("/login.do")
    public ModelAndView login(User user, HttpSession session){
        int id = userService.login(user.getUsername(), user.getPassword());
        ModelAndView modelAndView = new ModelAndView();
        if (id!=-1){
            List<Integer> roleIds = roleService.findRoleId(id);
            session.setAttribute("roleIds",roleIds);
            session.setAttribute("user",user);
            modelAndView.setViewName("main");
        }else {
            modelAndView.setViewName("../failer");
        }
        return modelAndView;
    }
    @RequestMapping("/findAll.do")
    public ModelAndView findAll(@RequestParam(defaultValue = "1") int currentPage,String username,@RequestParam(defaultValue = "0")int type,HttpSession session){
        if (type==1){
            session.setAttribute("searchname",username);
        }else {
            username=(String) session.getAttribute("searchname");
        }
        PageInfo<User> pageInfo = userService.findAll(currentPage,username);
        ModelAndView modelAndView = new ModelAndView();
        modelAndView.addObject("pageInfo",pageInfo);
        modelAndView.setViewName("user-list");
        return modelAndView;
    }
    @RequestMapping("/deleteById.do")
    public String delete(int id){
        userService.deleteById(id);
        return "redirect:findAll.do";

    }
    @RequestMapping("/add.do")
    public String add(User user){
        userService.add(user);
        return "redirect:findAll.do";
    }

    @RequestMapping("/toUpdate.do")
    public ModelAndView toUpdate(int id){
        User user = userService.selectUserById(id);
        ModelAndView modelAndView = new ModelAndView();
        modelAndView.setViewName("user-update");
        modelAndView.addObject("user",user);
        return modelAndView;
    }

    @RequestMapping("/update.do")
    public String update(User user){
        userService.update(user);
        return "redirect:findAll.do";
    }

    @RequestMapping("deleteAll.do")
    public String deleteAll(String userList){
        String[] strs = userList.split(",");
        List<Integer> ids = new ArrayList<>();
        for (String s:strs){
            ids.add(Integer.parseInt(s));
        }
        userService.deleteAll(ids);
        return "redirect:findAll.do";
    }

    @RequestMapping("toAddRole.do")
    public ModelAndView toAddRole(int id){
        List<Role> roleList = roleService.findRoleByUserId(id);
        ModelAndView mv = new ModelAndView();
        mv.addObject("roles",roleList);
        mv.addObject("id",id);
        mv.setViewName("user-role-add");
        return mv;
    }
    @RequestMapping
    @ResponseBody
    public String add(String roleList,String userId){
        String[] strs = roleList.split(",");
        List<Integer> ids=new ArrayList<>();
        for(String s:strs){
            ids.add(Integer.parseInt(s));
        }
        roleService.add(ids,userId);
        return "";
    }
}
```
## dao层
### UserDao

```java
package com.XuYijie.dao;

import com.XuYijie.bean.User;
import org.apache.ibatis.annotations.Param;

import java.util.List;

public interface UserDao {
    User findUserByUserName(String username);

    List<User> findAll(@Param("start") int start,@Param("username") String username);

    void deleteById(int id);

    void add(User user);

    User selectById(int id);

    void update(User user);

    int getTotalCount(@Param("username")String username);

    void deleteAll(@Param("ids") List<Integer> ids);
}

```
## RoleDao

```java
package com.XuYijie.dao;

import com.XuYijie.bean.Role;
import com.XuYijie.bean.UserRole;

import java.util.List;

public interface RoleDao {
    List<Integer> findRoleIdByUserId(int userid);

    List<Role> findRoleByUserId(int id);

    void addRole(UserRole userRole);
}

```

## service层
new pageinfo 对象	 给pageinfo 赋值
### IUserService

```java
package com.XuYijie.service;

import com.XuYijie.bean.PageInfo;
import com.XuYijie.bean.User;

import java.util.List;

public interface IUserService {
    boolean login(String username,String password);

    PageInfo<User> findAll(int currentPage,String username);

    void deleteById(int id);

    void add(User user);

    User selectUserById(int id);

    void update(User user);

    void deleteAll(List<Integer> ids);
}

```
### IRoleService

```java
package com.XuYijie.service;

import com.XuYijie.bean.Role;

import java.util.List;

public interface IRoleService {
    List<Integer> findRoleId(int userId);

    List<Role> findRoleByUserId(int id);

    void add(List<Integer> ids, String userId);
}

```

### impl包下的UserService

```java
package com.XuYijie.service.impl;

import com.XuYijie.bean.PageInfo;
import com.XuYijie.bean.User;
import com.XuYijie.dao.UserDao;
import com.XuYijie.service.IUserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;

@Service
public class UserService implements IUserService {
    @Autowired
    private UserDao userDao;
    @Override
    public boolean login(String username, String password) {
        User user = userDao.findUserByUserName(username);
        if (user != null&&user.getPassword().equals(password)){
            return true;
        }
        return false;
    }
    @Override
    public PageInfo<User> findAll(int currentPage,String username){
        PageInfo<User> pageInfo=new PageInfo<>();
        pageInfo.setSize(5);


        int tc = userDao.getTotalCount(username);
        pageInfo.setTotalCount(tc);
        int tp = (int)Math.ceil(tc/5.0);
        pageInfo.setTotalPage(tp);
        if (currentPage<1){
            pageInfo.setCurrentPage(1);
        } else if (currentPage>tp) {
            pageInfo.setCurrentPage(tp);
        }else {
            pageInfo.setCurrentPage(currentPage);
        }
        int start = (pageInfo.getCurrentPage()-1)*5;
        List<User> userList=userDao.findAll(start,username);
        pageInfo.setList(userList);
        return pageInfo;
    }

    //    @Override
//    public List<User> findAll() {
//        return userDao.findAll();
//    }
    @Override
    public void deleteById(int id){
        userDao.deleteById(id);
    }

    @Override
    public void add(User user) {
        userDao.add(user);
    }

    @Override
    public User selectUserById(int id) {
        return userDao.selectById(id);
    }

    @Override
    public void update(User user) {
        userDao.update(user);
    }

    @Override
    public void deleteAll(List<Integer> ids) {
        userDao.deleteAll(ids);
    }
}
```
### impl包下的RoleService

```java
package com.XuYijie.service.impl;

import com.XuYijie.bean.Role;
import com.XuYijie.bean.UserRole;
import com.XuYijie.dao.RoleDao;
import com.XuYijie.service.IRoleService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;
@Service
public class RoleService implements IRoleService {

    @Autowired
    private RoleDao roleDao;
    @Override
    public List<Integer> findRoleId(int userId) {
        return roleDao.findRoleIdByUserId(userId);
    }

    @Override
    public List<Role> findRoleByUserId(int id) {
        return roleDao.findRoleByUserId(id);
    }

    @Override
    public void add(List<Integer> ids, String userId) {
        for(int roleId:ids){
            UserRole userRole=new UserRole();
            userRole.setUserId(Integer.parseInt(userId));
            userRole.setRoleId(roleId);
            roleDao.addRole(userRole);
        }
    }
}

```

## fliter
在filter里面 判断session里是否有user  如果没有user并且当前的请求不是login.do 跳转到登入页面否则继续执行
步骤：
a.新建 loginfilter implements Filter
在filter里面 判断session里是否有user  如果没有user并且当前的请求不是login.do 跳转到登入页面
否则继续执行
b.web.xml  配置filter  拦截所有的*.do
c.到userController里的login 方法，登入成功后把用户信息放到session里面

### LoginFliter

```java
package com.XuYijie.filter;

import com.XuYijie.bean.User;

import javax.servlet.*;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;
import java.io.IOException;


public class LoginFilter implements Filter {
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {

    }

    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        HttpServletRequest request = (HttpServletRequest) servletRequest;
        HttpServletResponse response = (HttpServletResponse) servletResponse;
        HttpSession session = request.getSession();
        User user = (User) session.getAttribute("user");
        String uri = request.getRequestURI();
        if (user==null && uri.indexOf("login.do")==-1){
            response.sendRedirect(request.getContextPath()+"login.jsp");
        }else {
            filterChain.doFilter(request,response);
        }
    }

    @Override
    public void destroy() {

    }

}

```

## 配置Tomcat
[Tomcat下载地址](https://tomcat.apache.org/download-90.cgi)
选择Core里面的最后一个下载安装
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/Maven10.png)
在idea右上角点击“Add Configuration”
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/Maven11.png)
在左上角“加号”里面选择Tomcat Service下的Local
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/Maven12.png)
然后点右边“+”“Artifact”添加一个maven:war即可
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/Maven13.png)
# 项目展示
保持MySQL和数据库的连接状态
运行程序
等待运行完成
在浏览器地址栏中输入 localhost:8080/  回车
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/Maven14.png)
即可进入我们的登录页面，输入数据库里面的任意一个username和password
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/Maven15.png)
页面会显示我们数据库里的数据，并可以在页面上直接进行添加修改和删除
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/Maven0.png)

#  再放一次源码，以免你们没看到

不需要积分，直接下载


[Mave前后端的数据展示与操作源码](https://download.csdn.net/download/qq_48922459/52839676)
