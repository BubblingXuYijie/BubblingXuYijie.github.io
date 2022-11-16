---
title: Springboot 使用 SaToken 进行登录认证、权限管理以及路由规则接口拦截
date: 2022-09-26 11:26:43
categories: Java
tags:
    - SpringBoot
    - SaToken
cover: /img/SaTokenLogo.jpg
---
# 前言
> Sa-Token 是一个轻量级 Java 权限认证框架，主要解决：登录认证、权限认证、单点登录、OAuth2.0、分布式Session会话、微服务网关鉴权 等一系列权限相关问题。
><br>
>还有踢人下线、账号封禁、路由拦截规则、微服务网关鉴权、密码加密等丰富功能
>它不比 Shiro 和 SpringSecurity 的功能少，而且配置使用更加简单

---



# 一、引入和配置
先给你们看一下 Demo 文件结构

![在这里插入图片描述](https://img-blog.csdnimg.cn/da2fc49f138a4968a6fa6f1d6e7798d9.png)


## 1.引入依赖

> 如果不需要将 token 信息存入 redis，只需要引入下面这一个依赖

```xml
<dependency>
    <groupId>cn.dev33</groupId>
    <artifactId>sa-token-spring-boot-starter</artifactId>
    <version>1.31.0</version>
</dependency>
```

> 如果需要将 token 存入 redis，则还需要引入下面的依赖（一般搭建单点登录服务器才需要使用 redis）

`使用redis ，无需任何其他配置，只需要多引入下面几个依赖，然后下面的 yml 加一些配置，satoken 就可以自动存储到 redis，非常方便`

```xml
<!-- Sa-Token 整合 Redis （使用 jackson 序列化方式）-->
<dependency>
    <groupId>cn.dev33</groupId>
    <artifactId>sa-token-dao-redis-jackson</artifactId>
    <version>1.31.0</version>
</dependency>
<!-- Sa-Token插件：权限缓存与业务缓存分离 -->
<dependency>
    <groupId>cn.dev33</groupId>
    <artifactId>sa-token-alone-redis</artifactId>
    <version>1.31.0</version>
</dependency>
<!-- 提供Redis连接池 -->
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-pool2</artifactId>
</dependency>
```

---


## 2、配置yml
> 如下代码，如果不需要使用 redis ，则删除 `alone-redis`和`spring redis`配置，否则连接不到 redis 会报错

> 如果使用了 redis，我下面的配置是业务和鉴权分离的方式，也就是说，token 存储在 `alone-redis` 里面配置的数据库，我这里配置的是 `0` 号数据库，它和 `spring reids` 配置的数据库不冲突

```yaml
server:
  port: 8081
# Sa-Token配置
sa-token:
  # token前缀
  # Token前缀 与 Token值 之间必须有一个空格。
  # 一旦配置了 Token前缀，则前端提交 Token 时，必须带有前缀，否则会导致框架无法读取 Token。
  # 由于Cookie中无法存储空格字符，也就意味配置 Token 前缀后，Cookie 鉴权方式将会失效，此时只能将 Token 提交到header里进行传输
  # token-prefix: Bearer
  # token 名称 (同时也是cookie名称)
  token-name: satoken
  # token 有效期，单位s 默认30天, -1代表永不过期
  timeout: 2592000
  # token 临时有效期 (指定时间内无操作就视为token过期) 单位: 秒
  activity-timeout: -1
  # 是否允许同一账号并发登录 (为true时允许一起登录, 为false时新登录挤掉旧登录)
  is-concurrent: true
  # 在多人登录同一账号时，是否共用一个token (为true时所有登录共用一个token, 为false时每次登录新建一个token)
  is-share: false
  # token风格
  token-style: uuid
  # 是否输出操作日志
  is-log: true
  # 配置 Sa-Token 单独使用的 Redis 连接
  alone-redis:
    # Redis数据库索引（默认为0）
    database: 0
    # Redis服务器地址
    host: 127.0.0.1
    # Redis服务器连接端口
    port: 6379
    # Redis服务器连接密码（默认为空）
    password:
    # 连接超时时间
    timeout: 10s
spring:
  # 配置业务使用的 Redis 连接
  redis:
    # Redis数据库索引（默认为0）
    database: 1
    # Redis服务器地址
    host: 127.0.0.1
    # Redis服务器连接端口
    port: 6379
    # Redis服务器连接密码（默认为空）
    password:
    # 连接超时时间
    timeout: 10s

```
---

## 3、配置全局异常处理
> 这一步可以不配置，配置的作用是，在鉴权失败的时候，不会报错，而是返回给前端鉴权失败的原因，方便我们开发调试

> 下面的异常会在鉴权失败的时候自动返回到前端，无需我们手动抛出和返回

```java
package pers.xuyijie.satokendemo.exception;

import cn.dev33.satoken.util.SaResult;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;

/**
 * @author 徐一杰
 * @date 2022/9/23 16:45
 * @description SaToken全局异常拦截
 */
@RestControllerAdvice
public class GlobalExceptionHandler {

    /**
     * 全局异常拦截，鉴权失败不会报错，会返回给前端报错原因
      * @param e
     * @return
     */
    @ExceptionHandler
    public SaResult handlerException(Exception e) {
        e.printStackTrace();
        return SaResult.error(e.getMessage());
    }

}

```
---

## 4、模拟用户角色和权限

> 这里我们给用户分配一下我们模拟的角色和权限，正常你们要从数据库读取用户的角色和拥有的权限

> 这里实现了 StpInterface 下面的方法，下面的方法会在接口鉴权之前自动调用，判断角色和权限，无需我们手动调用

```java
package pers.xuyijie.satokendemo.permission;

import cn.dev33.satoken.stp.StpInterface;
import org.springframework.stereotype.Component;

import java.util.ArrayList;
import java.util.List;

/**
 * @author 徐一杰
 * @date 2022/9/23 16:46
 * @description 获取当前账号的权限和角色列表，这个类下面的方法会在接口鉴权之前自动调用
 */
@Component
public class UserPermission implements StpInterface {

    /**
     * 返回一个账号所拥有的权限码集合
     * 即你在调用 StpUtil.login(id) 时写入的标识值。
     */
    @Override
    public List<String> getPermissionList(Object loginId, String loginType) {
        // 本list仅做模拟，实际项目中要根据具体业务逻辑来查询权限
        List<String> list = new ArrayList<>();
        list.add("1");
        list.add("user-add");
        list.add("user-delete");
        list.add("user-update");
        list.add("user-get");
        list.add("article-get");
        System.out.println("用户权限列表：" + list);
        return list;
    }

    /**
     * 返回一个账号所拥有的角色标识集合 (权限与角色可分开校验)
     */
    @Override
    public List<String> getRoleList(Object loginId, String loginType) {
        // 本list仅做模拟，实际项目中要根据具体业务逻辑来查询角色
        List<String> list = new ArrayList<>();
        list.add("user");
        list.add("admin");
        list.add("super-admin");
        System.out.println("用户角色列表：" + list);
        return list;
    }

}

```
---

## 5、配置拦截器
> 如果在高版本 SpringBoot (≥2.6.x) 下注册拦截器失效，则需要添加 @EnableWebMvc 注解才可以使用

> 下面我们配置的规则叫作`路由拦截规则`，`/user/**` 意思就是接口地址为 `/user` 开头的所有接口，也就是说，下面的我们 `UserController` 里面的所有接口都在拦截范围内

```java
package pers.xuyijie.satokendemo.config;

import cn.dev33.satoken.config.SaTokenConfig;
import cn.dev33.satoken.interceptor.SaInterceptor;
import cn.dev33.satoken.router.SaRouter;
import cn.dev33.satoken.stp.StpUtil;
import org.springframework.boot.SpringBootConfiguration;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Primary;
import org.springframework.web.servlet.config.annotation.EnableWebMvc;
import org.springframework.web.servlet.config.annotation.InterceptorRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

/**
 * @author 徐一杰
 * @date 2022/9/23 16:49
 * @description
 */
@SpringBootConfiguration
@EnableWebMvc
public class SaTokenConfigure implements WebMvcConfigurer {

    /**
     * 注册 Sa-Token 拦截器，打开注解式鉴权功能
     * 如果在高版本 SpringBoot (≥2.6.x) 下注册拦截器失效，则需要额外添加 @EnableWebMvc 注解才可以使用
     * @param registry
     */
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        // 注册路由拦截器，自定义认证规则
        registry.addInterceptor(new SaInterceptor(handler -> {
            // 登录认证 -- 拦截所有路由，并排除/user/doLogin 用于开放登录
            SaRouter.match("/**", "/user/doLogin", r -> StpUtil.checkLogin());
            // 角色认证 -- 拦截以 admin 开头的路由，必须具备 admin 角色或者 super-admin 角色才可以通过认证
            SaRouter.match("/admin/**", r -> StpUtil.checkRoleOr("admin", "super-admin"));
            // 权限认证 -- 不同模块认证不同权限
            SaRouter.match("/user/**", r -> StpUtil.checkRole("user"));
            SaRouter.match("/admin/**", r -> StpUtil.checkPermission("admin"));
            // 甚至你可以随意的写一个打印语句
            SaRouter.match("/**", r -> System.out.println("--------权限认证成功-------"));
        }).isAnnotation(true))
        //拦截所有接口
        .addPathPatterns("/**")
        //不拦截/user/doLogin登录接口
        .excludePathPatterns("/user/doLogin");
    }

}

```
---

## 6、controller里调用satoken的方法
> 方法上面的注解是使用权限认证和拦截器的时候用的，下面我会讲到

> 我在下面的`UserController`演示了`登录`、`注销`、`检查是否登录`、`查看用户token`、`获取token有效期`、`对称加密`、`非对称加密`方法，具体的方法每一行代码的作用，都在注视中写出来了，等一下我们测试每一个方法，为大家展示运行结果并解析代码

```java
package pers.xuyijie.satokendemo.controller;

import cn.dev33.satoken.annotation.SaIgnore;
import cn.dev33.satoken.basic.SaBasicUtil;
import cn.dev33.satoken.secure.SaBase64Util;
import cn.dev33.satoken.secure.SaSecureUtil;
import cn.dev33.satoken.stp.SaLoginModel;
import cn.dev33.satoken.stp.SaTokenInfo;
import cn.dev33.satoken.stp.StpUtil;
import cn.dev33.satoken.util.SaResult;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.HashMap;

/**
 * @author 徐一杰
 * @date 2022/9/23 15:52
 * @description
 */
@RestController
@RequestMapping("/user")
public class UserController {

    private static final String USERNAME = "xyj";
    private static final String PASSWORD = "123456";

    /**
     * 测试登录
     * @param username
     * @param password
     * @return
     */
    @RequestMapping("/doLogin")
    public SaResult  doLogin(String username, String password) {
        //这个方法会强制在浏览器弹出一个认证框
        SaBasicUtil.check("sa:123456");
        if(username.equals(USERNAME) && password.equals(PASSWORD)) {
            //这个是登录用户的主键，业务中你要从数据库中读取
            StpUtil.login(1);
            //获取登录生成的token
            tokenInfo = StpUtil.getTokenInfo();
            System.out.println(tokenInfo);
            return SaResult.ok("登录成功，会话ID为 " + StpUtil.getLoginId() + " ，Token为：" + StpUtil.getTokenValue());
        }
        return SaResult.error("登录失败");
    }

    /**
     * 查询登录状态
     * @return
     */
    @RequestMapping("/signOut")
    public SaResult signOut() {
        String loginId = null;
        if (StpUtil.isLogin()){
            loginId = (String) StpUtil.getLoginId();
            StpUtil.logout();
        }
        return SaResult.ok("会话ID为 " + loginId + " 的用户注销登录成功");
    }

    /**
     * 查询登录状态
     * @return
     */
    @RequestMapping("/isLogin")
    public SaResult isLogin() {
        if (StpUtil.isLogin()){
            return SaResult.ok("会话是否登录：" + StpUtil.isLogin() + " ，会话ID为 " + StpUtil.getLoginId());
        }
        return SaResult.ok("会话是否登录：" + StpUtil.isLogin());
    }

    /**
     * 根据Token值获取对应的账号id，如果未登录，则返回 null
     * @param tokenValue
     * @return
     */
    @RequestMapping("/getUserByToken/{tokenValue}")
    public SaResult getUserByToken(@PathVariable String tokenValue){
        return SaResult.ok((String) StpUtil.getLoginIdByToken(tokenValue));
    }

    /**
     * 获取当前会话剩余有效期（单位：s，返回-1代表永久有效）
     * @return
     */
    @RequestMapping("/getTokenTimeout")
    public SaResult getTokenTimeout(){
        return SaResult.ok(String.valueOf(StpUtil.getTokenTimeout()));
    }

    @SaIgnore
    @RequestMapping("/encodePassword")
    public void encodePassword() throws Exception {
        /**
         * md5加盐加密: md5(md5(str) + md5(salt))
         */
        String md5 = SaSecureUtil.md5("123456");
        String md5BySalt = SaSecureUtil.md5BySalt("123456", "salt");
        System.out.println("MD5加密：" + md5);
        System.out.println("MD5加盐加密：" + md5BySalt);

        /**
         * AES对称加密
         */
        // 定义秘钥和明文
        String key = "123456";
        String text = "这是一个明文用于测试AES对称加密";
        // 加密
        String ciphertext = SaSecureUtil.aesEncrypt(key, text);
        System.out.println("AES加密后：" + ciphertext);
        // 解密
        String text2 = SaSecureUtil.aesDecrypt(key, ciphertext);
        System.out.println("AES解密后：" + text2);

        /**
         * RSA非对称加密
         */
        // 定义私钥和公钥
        HashMap<String, String> keyMap = SaSecureUtil.rsaGenerateKeyPair();
        String privateKey = keyMap.get("private");
        String publicKey = keyMap.get("public");
        // 文本
        String text1 = "这是一个明文用于测试RSA非对称加密";
        // 使用公钥加密
        String ciphertext1 = SaSecureUtil.rsaEncryptByPublic(publicKey, text1);
        System.out.println("公钥加密后：" + ciphertext1);
        // 使用私钥解密
        String text3 = SaSecureUtil.rsaDecryptByPrivate(privateKey, ciphertext1);
        System.out.println("私钥解密后：" + text3);

        /**
         * Base64
         */
        // 文本
        String text4 = "这是一个明文用于测试Base64";
        // 使用Base64编码
        String base64Text = SaBase64Util.encode(text4);
        System.out.println("Base64编码后：" + base64Text);
        // 使用Base64解码
        String text5 = SaBase64Util.decode(base64Text);
        System.out.println("Base64解码后：" + text5);

    }

}

```
> 下面的`TestController`里面等下演示权限认证和路由拦截的时候用

```java
package pers.xuyijie.satokendemo.controller;

import cn.dev33.satoken.annotation.*;
import cn.dev33.satoken.util.SaResult;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

/**
 * @author 徐一杰
 * @date 2022/9/23 16:38
 * @description
 */
@RestController
@RequestMapping("/test")
@SaCheckLogin
public class TestController {

    /**
     * 此接口加上了 @SaIgnore 可以游客访问
      * @return
     */
    @SaIgnore
    @RequestMapping("/getList")
    public SaResult getList() {
        return SaResult.ok("无需登录接口");
    }

    /**
     * 登陆后才可调用该方法
     * @return
     */
    @SaCheckLogin
    @RequestMapping("/select")
    public SaResult select(){
        return SaResult.ok("查询成功");
    }

    /**
     * 必须具有指定权限才能进入该方法
     * @return
     */
    @SaCheckRole("super-admin")
    @RequestMapping("/delete")
    public SaResult delete() {
        return SaResult.ok("删除成功");
    }

    /**
     * 注解式鉴权：SaMode.OR 只要具有其中一个权限即可通过校验
     * @return
     */
    @RequestMapping("/add")
    @SaCheckPermission(value = {"user-add", "user-all"}, mode = SaMode.OR)
    public SaResult add() {
        return SaResult.ok("添加成功");
    }

    /**
     * 一个接口在具有权限 user-update 或角色 admin 时可以调通
     * @return
     */
    @RequestMapping("/update")
    @SaCheckPermission(value = "user-add", orRole = "admin")
    public SaResult update() {
        return SaResult.ok("更新成功");
    }

    /**
     * 这个接口测试用
     * @return
     */
    @RequestMapping("/testPermission")
    @SaCheckPermission(value = "user123")
    public SaResult testPermission() {
        return SaResult.ok("这个接口测试用");
    }

}

```


---
# 二、登录演示
`到这里，我么前期的配置就已经结束了，下面我开始测试每一个方法，为大家展示运行结果并解析代码，先把项目运行起来`

![在这里插入图片描述](https://img-blog.csdnimg.cn/e76170e24e6c4e42b31086c7afdb0480.png)


## 1、登录-doLogin
> 大家请看，我在请求 `/doLogin` 这个接口的时候，弹出了下面的认证框，这个就是方法第一行代码的功能，这叫 `Basic认证`，当然可以不要这一行代码，随你们，认证账号 sa 密码 123456

```java
//这个方法会强制在浏览器弹出一个认证框
SaBasicUtil.check("sa:123456");
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/dc97a725158b452aa9f9ba5c4a76cb16.png)
> 进行 `Basic认证` 后，登陆成功
![在这里插入图片描述](https://img-blog.csdnimg.cn/3fc2ac5572a348f1bab2f068879fe5a9.png)

`我这里使用了 redis ，token 已经存储进来了`
![在这里插入图片描述](https://img-blog.csdnimg.cn/5060b4810cd348a09ea73e1eb7410b20.png)


---

## 2、验证登录-isLogin

![在这里插入图片描述](https://img-blog.csdnimg.cn/b6b9cb9ddd0747e3bfa1e49d58df19e7.png)

---

## 3、获取token时效-getTokenTimeout
> 这是我们在 yml 里面配置的时效性，30天的毫秒
![在这里插入图片描述](https://img-blog.csdnimg.cn/ffc87c8d8a164ead92e8380a77c200c2.png)

---

## 4、加密
> 请求 `encodePassword` 接口

![在这里插入图片描述](https://img-blog.csdnimg.cn/0537dab8801c4133841b05c36e13acc5.png)

---


## 5、注销登录-logout

![在这里插入图片描述](https://img-blog.csdnimg.cn/6fdd6923346342e490075ca757151298.png)

---

# 三、权限认证和拦截器演示
> 下面演示上面我们配置的拦截器，satoken 可以直接使用注解来进行拦截，很方便

> 我们可以发现，我在两个 controller 里面使用了 satoken 的几个注解，注解可以用在方法上或类上
> `@SaIgnore` 忽略该方法，不进行任何拦截和鉴权
> `@SaCheckLogin` 登录后才可以调用该接口
> `@SaCheckRole("super-admin")` 登录用户必须要是"super-admin"角色才可调用
> `@SaCheckPermission(value = "del")` 登录用户必须要有"del"权限才可调用

## 1、登录认证
> 下面我们调用添加了 `@SaCheckLogin` 注解的方法
### (1) 未登录情况
![在这里插入图片描述](https://img-blog.csdnimg.cn/32b3a5c2545643e6a58d5facc6f16d97.png)
### (2) 已登陆情况

> 可以看到调用成功，控制台打印出我们上面拦截器配置的输出信息
>
![在这里插入图片描述](https://img-blog.csdnimg.cn/ad17714448e34d408af24b2fb4550eb3.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/967ba4056be749f4b3ec707eed0641c3.png)

---

## 2、权限认证
> 登录后，我们调用增加了 `@SaCheckPermission(value = "del")` 注解的方法，可以看到提示`无此权限：del`，因为前面我们模拟用户权限时，没有给用户分配 `del` 权限

![在这里插入图片描述](https://img-blog.csdnimg.cn/86aaf82cafc14130be209cd4da0b80c5.png)
> 我们再调用增加了 `@SaCheckRole("super-admin")` 注解的方法，可以看到成功

![在这里插入图片描述](https://img-blog.csdnimg.cn/85c641e9ee53498caa548115295e53a0.png)

---

# 总结
