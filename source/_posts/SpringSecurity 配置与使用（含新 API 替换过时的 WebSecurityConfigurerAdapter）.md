---
title: SpringSecurity 配置与使用（含新 API 替换过时的 WebSecurityConfigurerAdapter）
date: 2022-10-28 11:13:27
categories: Java
tags: 
    - SpringBoot
    - SpringSecurity
cover: https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/springbootLogo.jpeg
---
# 前言
As we all know，现今主流权限框架有 SpringSecurity、Shiro、SaToken，Shiro在前后端分离时代基本被淘汰，剩下适合大型项目的 `SpringSecurity` 和 适合中小型项目的 `SaToken` 可以选择，SaToken 我也写了文章 [Springboot 使用 SaToken 进行登录认证、权限管理以及路由规则接口拦截](https://blog.csdn.net/qq_48922459/article/details/127048667?spm=1001.2014.3001.5501)

> SpringSecurity 作为 Spring 的官方权限框架，肯定是最牛逼的，当然也最复杂，中小型项目还是 SaToken 来的省心呀，简单，几行代码实现认证、拦截、踢人、单点登录等，SpringSecurity 想要实现这些功能，需要深入研究，现在我只写最简单的用户认证和接口权限控制。

`现在的 SpringSecurity 版本更换了新的配置方式，下面有写`

---


# 一、引入依赖
> 自己新建一个标准的 Springboot web 项目，然后增加下面这个依赖

```xml
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```
---

# 二、启动类增加注解

> `@EnableWebSecurity`表示启用户 springsecurity 功能
> `@EnableGlobalMethodSecurity(prePostEnabled = true)`是开启基于注解的接口权限控制

```java
/**
 * @author 徐一杰
 * @date 2022/10/25 17:18
 * 开启方法级安全验证
 */
@SpringBootApplication
@EnableWebSecurity
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class SpringSecurityDemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringSecurityDemoApplication.class, args);
    }

}
```

---


# 三、config配置文件
`现在的 SpringSecurity 版本更换了新的配置方式，目前新版本仍兼容旧版配置，你不喜欢新版配置也可以用旧版`

## 旧版配置

```java
import org.springframework.boot.SpringBootConfiguration;
import org.springframework.context.annotation.Bean;
import org.springframework.http.HttpMethod;
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.builders.WebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;

/**
 * @author 徐一杰
 * @date 2022/10/25 15:28
 * 这是旧版api
 */
@SpringBootConfiguration
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {

    /**
     * 指定加密方式
     */
    @Bean
    public PasswordEncoder passwordEncoder(){
        // 使用官方推荐的BCrypt加密密码，也就是md5加随机盐
        return new BCryptPasswordEncoder();
    }

    /**
     * configure(WebSecurity)用于影响全局安全性(配置资源，设置调试模式，通过实现自定义防火墙定义拒绝请求)的配置设置。
     * 一般用于配置全局的某些通用事物，例如静态资源等
     * @param web 
     * @throws Exception
     */
    public void configure(WebSecurity web) throws Exception {
        web.ignoring().antMatchers("/resources/**", "/ignore2");
    }

    /**
     * 配置接口拦截
     * configure(HttpSecurity)允许基于选择匹配在资源级配置基于网络的安全性，也就是对角色所能访问的接口做出限制
     * @param httpSecurity 请求属性
     * @throws Exception
     */
    @Override
    protected void configure(HttpSecurity httpSecurity) throws Exception {
        httpSecurity.authorizeRequests()
                // 允许get请求/test/any，而无需认证，不配置HttpMethod默认允许所有请求类型
                .antMatchers(HttpMethod.GET, "/test/any").permitAll()
                //指定权限为admin才能访问，这里和方法注解配置效果一样，但是会覆盖注解
                .antMatchers("/test/admin").hasRole("admin")
                // 所有请求都需要验证
                .anyRequest().authenticated()
                .and()
                //.httpBasic() Basic认证，和表单认证只能选一个
                // 使用表单认证页面
                .formLogin()
                //配置登录入口，默认为security自带的页面/login
                .loginProcessingUrl("/login")
                .and()
                // post请求要关闭csrf验证,不然访问报错；实际开发中开启，需要前端配合传递其他参数
                .csrf().disable();
    }
}

```


---

## 新版配置

```java
package icu.xuyijie.springsecuritydemo.config;

import org.springframework.boot.SpringBootConfiguration;
import org.springframework.context.annotation.Bean;
import org.springframework.http.HttpMethod;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityCustomizer;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.security.web.SecurityFilterChain;

/**
 * @author 徐一杰
 * @date 2022/10/25 17:34
 * 新api
 */
@SpringBootConfiguration
public class WebSecurityNewConfig {

    /**
     * 指定加密方式
     */
    @Bean
    public PasswordEncoder passwordEncoder(){
        // 使用官方推荐的BCrypt加密密码，也就是md5加随机盐
        return new BCryptPasswordEncoder();
    }

    /**
     * configure(WebSecurity)用于影响全局安全性(配置资源，设置调试模式，通过实现自定义防火墙定义拒绝请求)的配置设置。
     * 一般用于配置全局的某些通用事物，例如静态资源等
     * 新版本其实不推荐把路径的拦截写在这里，而是推荐写在securityFilterChain里面
     * "You are asking Spring Security to ignore Ant [pattern='/resources/**']. This is not recommended -- please use permitAll via HttpSecurity#authorizeHttpRequests instead"
     */
    @Bean
    public WebSecurityCustomizer webSecurityCustomizer() {
        return (web) -> web.ignoring().antMatchers("/resources/**", "/ignore2");
    }

    /**
     * 配置接口拦截
     * configure(HttpSecurity)允许基于选择匹配在资源级配置基于网络的安全性，也就是对角色所能访问的接口做出限制
     * @param httpSecurity 请求属性
     * @return HttpSecurity
     * @throws Exception
     */
    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity httpSecurity) throws Exception {
        httpSecurity.authorizeRequests()
                // 允许get请求/test/any，而无需认证，不配置HttpMethod默认允许所有请求类型
                .antMatchers(HttpMethod.GET, "/test/any", "/js/**", "/css/**", "/images/**", "/icon/**", "/file/**").permitAll()
                // 指定权限为admin才能访问，这里和方法注解配置效果一样，但是会覆盖注解
                .antMatchers("/test/admin").hasRole("admin")
                // 所有请求都需要验证
                .anyRequest().authenticated()
                .and()
                //.httpBasic() Basic认证，和表单认证只能选一个
                // 使用表单认证页面
                .formLogin()
                // 配置登录入口，默认为security自带的页面/login
                .loginProcessingUrl("/login")
                .and()
                // post请求要关闭csrf验证,不然访问报错；实际开发中开启，需要前端配合传递其他参数
                .csrf().disable();
        return httpSecurity.build();
    }
}

```

---

# 四、UserDetailsServiceImpl

> `UserDetailsService` 是 SpringSecurity 的内置类，我们需要实现它的 loadUserByUsername 方法，方法参数 `username` 就是登录时填写的用户名，里面写从数据库获取这个 username 的密码和角色，然后 return 给 `SpringSecurity 内置的 User 实体类`


```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.stereotype.Service;

import java.util.ArrayList;
import java.util.List;

/**
 * @author 徐一杰
 * @date 2022/10/25 15:22
 */
@Service
public class UserDetailsServiceImpl implements UserDetailsService {
	/**
     * 注入SpringSecurity内置的密码加密类
     */
	@Autowired
    private final PasswordEncoder passwordEncoder;

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        // 通过用户名从数据库获取用户信息，略，我们直接手动定义个用户
        // 定义用户角色为 user
        String role = "user";
        // 定义用户密码
        String password = "123456";
        // 角色列表
        List<GrantedAuthority> authorityList = new ArrayList<>();
        // 角色必须以ROLE_开头，security在判断角色时会自动截取ROLE_
        authorityList.add(new SimpleGrantedAuthority("ROLE_" + role));

        return new User(
                username,
                // springsecurity对比密码时默认会解密密码后对比，因为数据库一般存取加密后的密码，我们这里是明文，所以需加密密码
                passwordEncoder.encode(password),
                authorityList
        );
    }
}

```

---

# 五、写一个Controller测试用
> 上面主启动类添加的注解开启基于注解的接口权限的意思就是开启下面的 `@PreAuthorize`注解的功能，
> 这个注解和`config` 里面配置的 `.antMatchers("/test/admin").hasRole("admin")` 一个意思，选其中一个方式即可。

```java
import org.springframework.security.access.prepost.PreAuthorize;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

/**
 * @author 徐一杰
 * @date 2022/10/25 16:28
 */
@RestController
@RequestMapping("/test")
public class TestController {

    /**
     * user和admin角色能访问该方法
     */
    @PreAuthorize("hasAnyRole('user', 'admin')")
    @RequestMapping("/user")
    public String user(){
        System.out.println("user和admin角色访问");
        return "user和admin角色访问";
    }

    /**
     * admin角色才能访问该方法
     */
    @PreAuthorize("hasAnyRole('admin')")
    @RequestMapping("/admin")
    public String admin(){
        System.out.println("admin角色访问");
        return "admin角色访问";
    }

    /**
     * WebSecurityConfig里配置了该接口Get请求无需登录
     */
    @RequestMapping("/any")
    public String any(){
        System.out.println("这个接口Get请求无需登录");
        return "这个接口Get请求无需登录";
    }

}

```

---

# 六、启动测试

> 我们先访问 `http://127.0.0.1:8081/test/any`，这时我们还没有登录，这个接口我们在`config` 里面配置了`.permitAll()`，所以没有被拦截，直接访问成功

![在这里插入图片描述](https://img-blog.csdnimg.cn/d64a9859d0174f2fb7d05e0f987651ee.png)
> 下面我门访问 `http://127.0.0.1:8081/test/user`，发现浏览器自动跳转到了登录界面，这个登录界面是SpringSecurity内置的，如需使用自定义页面，下面会讲
>
![在这里插入图片描述](https://img-blog.csdnimg.cn/2c3a050987ce4b8bbcd65c14ab2ece4c.png)
> 输入账号密码点击 Sign in，发现浏览器自动跳回`http://127.0.0.1:8081/test/user`，访问成功
> 这里我们用户名可以随便输入，因为上面`UserDetailsServiceImpl`中我们没有指定用户名，所以 123456 这个密码所有用户都能用

![在这里插入图片描述](https://img-blog.csdnimg.cn/85b9b64cdf934e14a8ba899e4f2297cb.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/78b40ea88cf94802bd659a65dea0ec79.png)

> 如果访问 `/test/admin` 这个接口，报错 403，代表无权限
>
![在这里插入图片描述](https://img-blog.csdnimg.cn/534b496df8e1431e970cf6a6ad70e7a1.png)


---

# 七、前后端分离设计
## 1、自定义登录界面
> `config`里面配置的 `.loginProcessingUrl("/login")`是默认使用SpringSecurity内置登录页面，如果需要使用前端登陆页面，可以配置一个 MvnConfig 拦截接口，让前端跳转到他们的登录页面，然后把登录请求发送给/login这个内置接口就行了

> 当然也可以单独写一个登录页面放到后端的 `resources/static`里面，这样可以直接在 `config的loginProcessingUrl`中修改

## 2、自定义登录成功/失败处理器
> 自定义处理器，这里写你登录失败的逻辑，返回给前端数据，让前端进行页面跳转

```java
import org.springframework.security.core.AuthenticationException;
import org.springframework.security.web.authentication.SimpleUrlAuthenticationFailureHandler;
import org.springframework.stereotype.Component;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

/**
 * @author 徐一杰
 * @date 2022/10/25 17:19
 * security认证失败逻辑，定义这个以后，security内置的页面跳转就失效了
 */
@Component
public class LoginFailureHandler extends SimpleUrlAuthenticationFailureHandler {

    @Override
    public void onAuthenticationFailure(HttpServletRequest request, HttpServletResponse response, AuthenticationException exception) throws IOException, ServletException {
        response.setContentType("application/json;charset=UTF-8");
        System.out.println("登录成功");
        //这里写你登录失败的逻辑，返回给前端数据，让前端进行页面跳转
    }
}

```


> 在`config`的`securityFilterChain`方法里面添加下面代码，`loginSuccessHandler`代码和上面一样

```java
// 使用自定义的结果处理器，定义这个以后，security内置的页面跳转就失效了
      .successHandler(loginSuccessHandler)
      .failureHandler(loginFailureHandler)
```

---

# 总结


> SpringSecurity 作为 Spring 的官方权限框架，肯定是最牛逼的，当然也最复杂，中小型项目还是 SaToken 来的省心呀，简单，几行代码实现认证、拦截、踢人、单点登录等，SpringSecurity 想要实现这些功能，SaToken 我也写了文章 [Springboot 使用 SaToken 进行登录认证、权限管理以及路由规则接口拦截](https://blog.csdn.net/qq_48922459/article/details/127048667?spm=1001.2014.3001.5501)
