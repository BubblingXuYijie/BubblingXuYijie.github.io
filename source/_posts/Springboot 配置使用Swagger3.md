---
title: Springboot 配置使用Swagger3
date: 2022-03-12 22:21:09
categories: Java
tags:
    - SpringBoot
    - Swagger
cover: https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/Swagger0.png
---
# 前言

Swagger是一个可以根据你的代码，自动生成接口文档的一个工具，并且可以用作接口测试工具，2022年了，Swagger也要用3.0版本了吧


---


# 一、引入依赖
没错，Swagger 3.0版本只需要引入这一个依赖


```xml
<!-- https://mvnrepository.com/artifact/io.springfox/springfox-boot-starter -->
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-boot-starter</artifactId>
            <version>3.0.0</version>
        </dependency>
```

---

# 二、配置
在项目中新建 Swagger3Config


```java
package pers.xuyijie.communityinteractionsystem.config;

import io.swagger.annotations.ApiOperation;
import org.springframework.boot.SpringBootConfiguration;
import org.springframework.context.annotation.Bean;
import springfox.documentation.builders.ApiInfoBuilder;
import springfox.documentation.builders.PathSelectors;
import springfox.documentation.builders.RequestHandlerSelectors;
import springfox.documentation.oas.annotations.EnableOpenApi;
import springfox.documentation.service.ApiInfo;
import springfox.documentation.service.Contact;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;

/**
 * @Author: 徐一杰
 * @date: 2022/3/12
 * @Description: Swagger3配置文件
*/
@SpringBootConfiguration
@EnableOpenApi
public class Swagger3Config {
    /**
     *   application中还配置了mvc，因为springboot2.6.1与swagger3不兼容
     */

    public Docket createRestApi() {
        return new Docket(DocumentationType.OAS_30)
                .apiInfo(apiInfo())
                // ture 启用Swagger3.0， false 禁用（生产环境要禁用）
                .enable(false)
                .select()
                // 扫描的路径使用@Api的controller
                .apis(RequestHandlerSelectors.withMethodAnnotation(ApiOperation.class))
                // 指定路径处理PathSelectors.any()代表所有的路径
                .paths(PathSelectors.any())
                .build();
    }

    /**
     * API 页面上半部分展示信息
     */
    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                .title("Swagger3接口文档")
                .description("海里社区交互软件接口文档")
                .contact(new Contact("徐一杰", "https://xuyijie.icu/", "1119461672@qq.com"))
                .version("1.0")
                .build();
    }
}

```
---

#  三、配置 application.yml
如果你使用的是 Springboot 2.6 版本，需要配置，否则报下面的错，现在 Springboot 3.0 和 Springboot 2.5.8 不需要配置下面这个，你们看情况怎么办

`Caused by: java.lang.NullPointerException: Cannot invoke "org.springframework.web.servlet.mvc.condition.PatternsRequestCondition.getPatterns()" because "this.condition" is null`

```yaml
spring:
#  这个mvc的配置是springboot2.6.1不支持swagger3的折衷配置，后面考虑升级Springboot版本或降级版本
  mvc:
    pathmatch:
      matching-strategy: ant_path_matcher
```


---

#  四、使用方法
在 controller 上面增加 `@Api(tags = "发贴版块的社区交流和匿名举报")`注解，tags里面是你对这个 controller 的描述

在 controller 里面的方法上增加`@ApiOperation(value = "发布帖子")`注解，value是你对这个方法的描述




```java
/**
 * @Author: 徐一杰
 * @date: 2022/3/8
 * @Description: 社区交流、匿名举报
 */
@Api(tags = "发贴版块的社区交流和匿名举报")
@RestController
@RequestMapping("/blog")
public class BlogController {
    private final BlogService blogService;

    public BlogController(BlogService blogService) {
        this.blogService = blogService;
    }
    

    @PostMapping("/posted")
    @ApiOperation(value = "发布帖子")
    public ResultCode<Blog> posted(@RequestBody Blog blog, HttpServletRequest request) throws FileNotFoundException {
        return blogService.posted(blog,request);
    }
    
}
```

实体类也可以增加注解，也可以不加，主要是 controller 加就行

实体类上面增加`@ApiModel(value = "Blog", description = "社区交流、匿名举报帖子实体类")`，下面的字段增加`@ApiModelProperty(value = "主键")`，value 是 Swagger 生成的文档中显示的名字，description 是你对实体类的描述



```java
/**
 * @Author: 徐一杰
 * @date: 2022/3/8
 * @Description: 社区交流、匿名举报帖子实体类
 */
@Data
@TableName("blog")
@ApiModel(value = "Blog", description = "社区交流、匿名举报帖子实体类")
public class Blog implements Serializable {
    @ApiModelProperty(value = "主键")
    private String id;
    @ApiModelProperty(value = "所属社区")
    private String belongCommunity;
    
}
```



---




# 总结
启动项目，访问`http://localhost:8081/swagger-ui/`，注意 Swagger3 和 2 访问的页面有细微差别

![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/Swagger0.png)


