---
title: 记Linux使用宝塔部署Vue+Springboot前后端分离项目
date: 2021-12-13 20:09:58
categories: Linux
tags:
    - Linux
    - SpringBoot
    - 宝塔面板
cover: https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/宝塔部署0.png
---
<hr style=" border:solid; width:100px; height:1px;" color=#000000 size=1">


（已解决事务未注册和数据库连接超时问题、nginx转发后端接口问题，访问静态资源配置，配置获取登录用户设备所在的 IP）

# 前言

毕业设计，第一次部署到自己的服务器，记一下详细的教程还有过程中遇到的坑</font>

<font color=#999AAA >需要的详细后端打包教程和 Linux 服务器上传文件，连接服务器方法和命令，看我的另一篇很详细的，前端打包不需要教程，直接 npm run build 完事儿
[Java非Web的Springboot项目打包部署到Linux服务器并运行（Maven、Gradle）](https://editor.csdn.net/md/?articleId=121718064)


<font color=#999AAA >本篇也没有介绍如何从前端上传图片并使用后端处理后讲 base64 转化为 url 存入数据库，需要学习的传送门：[Vue+Springboot上传图片将 Base64 码转换为图片保存在指定文件夹](https://blog.csdn.net/qq_48922459/article/details/122054809)



<hr style=" border:solid; width:100px; height:1px;" color=#000000 size=1">


# 一、项目打包


将 Vue 项目里所有向后端请求的地址都修改为将要部署的服务器的 IP ，端口号可以不变，因为在服务器上使用 nginx 可以转发。

<font color=#999AAA >后端数据库连接修改成服务器的 IP 和端口（下面会教在服务器创建数据库），需要注意的是后端 Java 的打包，项目创建用的JDK版本要和服务器上一样，不然运行会报错；

<font color=#999AAA >数据库连接的 mysql-connect jar 包一定要和服务器上的 MySQL 环境的大版本相同，比如服务器上安装的是 MySQL 5.7，你就不能用 8.0.27 版本的 mysql-connect jar 包，否则会报错；

<font color=#999AAA >还有 application.yml 里数据库连接的配置 `jdbc:mysql://111.24.211.111:3306/xuyijie_icu?useSSL=false&useUnicode=true&characterEncoding=UTF-8` 里面的 `useSSL=false`  一定要设置成 false ，不然有概率连接数据库报错

<hr style=" border:solid; width:100px; height:1px;" color=#000000 size=1">

# 二、上传打包文件

## 1.Vue

<font color=#999AAA >首先在宝塔上新建站点和数据库，域名那里如果没有域名，就写自己的服务器IP，端口可以指定，但是不允许是8080，记得开放服务器和宝塔防火墙（==服务器和宝塔各有防火墙，都要开放，自己设置的网站端口要开放，数据库的3306要开放，根据需要）==，数据库可以选择创建或者以后再创建，提交

![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/宝塔部署1.png)

<font color=#999AAA >我以前创建好了点开网站根目录，初始有两张页面，我们可以把 index.html 和 404.html 删了，因为要放我们的 Vue 项目

![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/宝塔部署2.png)


<font color=#999AAA >直接把打包好的 dist 文件夹里面的东西，上传到这里就行了，里面的 ==java_back==文件夹是我自己创建的，里面放了 Springboot 打的包，后端项目放哪里其实无所谓，你们自己看着办

![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/宝塔部署3.png)

<hr style=" border:solid; width:100px; height:1px;" color=#000000 size=1">

## 1.Springboot



<font color=#999AAA >首先下载我给的文件，这是jar在linux上启动的通用脚本，不需要积分，一定要下，网上的 java jar 这种启动方式是暂时启动
[启动脚本和配置文件下载](https://download.csdn.net/download/qq_48922459/55643280)
<font color=#999AAA >下载完成后把解压出来的文件夹里的文件拖到服务器jar包所在的文件夹

![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/宝塔部署4.png)

![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/宝塔部署5.png)

<font color=#999AAA >建议用 **XShell** 连接你的数据库，`cd` 到 jar 包位置，输入 `ls` 查看文件，然后输入 `sh build.sh` 来运行这个文件，会显示构建成功，这时会发现多了一个autoScript文件夹，`cd autoScript`，里面有几个sh文件，`sh startup.sh` ，项目就会启动了，并且永远不会停止，除非你运行 `sh shutdown.sh`。OK，完成。放心的退出你的服务器吧。


如果 XShell 还有连接数据库、命令什么的不会，我的另一篇很详细，看我的传送门 [Java非Web的Springboot项目打包部署到Linux服务器并运行（Maven、Gradle）](https://editor.csdn.net/md/?articleId=121718064)

![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/宝塔部署0.png)
<hr style=" border:solid; width:100px; height:1px;" color=#000000 size=1">




# 三、配置Nginx反向代理

<font color=#999AAA >因为前后端项目运行在同一台服务器，所以必定端口不同，需要配置 Nginx 才能请求到后端


<font color=#999AAA >站点设置里的配置文件，添加我鼠标选中的东西，里面的 IP 和端口填你自己的服务器 IP 和后端设置的端口，==/api/== 是你 Vue 里面自己配置的 axios 的 baseURL 的转发地址，如果没有配置，把 ==/api/== 换成  /  就行了

![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/宝塔部署6.png)
```bash
	location /api/ {
      proxy_pass http://111.111.11.111:8081/;
    }
```

<hr style=" border:solid; width:100px; height:1px;" color=#000000 size=1">


# 四、配置静态资源（可以上传图片到服务器并访问）

要想网页上传图片到服务器指定文件夹并访问到服务器的图片，要配置两个东西，后端代码和服务器的Nginx。


##  1、Nginx配置静态资源代理
下面的配置的意思是说：

==http://你的IP:端口号/user_head/aa.png==可以访问到 ==/www/wwwroot/xuyijie.icu/upload_file/user_head/== 里面的aa.png

==http://你的IP:端口号/blog_img/aa.png==可以访问到 ==/www/wwwroot/xuyijie.icu/upload_file/blog_img/== 里面的aa.png




```html
	# 下面两个location的意思是访问 http://你的IP:端口号/user_head/aa.png 就可以访问到在 
	/www/wwwroot/xuyijie.icu/upload_file/user_head/ 目录下的aa图片了，可以配置多个 location
	#然后你就可以把http://你的IP:端口号/user_head/aa.png写到你的前端的 scr 里面就可以访问到了
	# ^~ 表示 如果把这个前缀用于一个常规字符串,那么告诉nginx 如果路径匹配那么不测试正则表达式。
	location ^~ /user_head/ {
		root /www/wwwroot/xuyijie.icu/upload_file/user_head/;     #指定图片存放路径    
	}
		
	location ^~ /user_head/ {
		root /www/wwwroot/xuyijie.icu/upload_file/user_head/;     #指定图片存放路径    
	}
```
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/宝塔部署7.png)


<hr style=" border:solid; width:100px; height:1px;" color=#000000 size=1">

##  2、Springboot配置物理地址映射

<font color=#999AAA >新建 config 包，在里面新建 WebConfig 类
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/宝塔部署8.png)
<font color=#999AAA >代码如下，解析都在注释里


```java
package pers.xuyijie.communityinteractionsystem.config;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.SpringBootConfiguration;
import org.springframework.web.servlet.config.annotation.ResourceHandlerRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;
@SpringBootConfiguration
public class WebConfigurer implements WebMvcConfigurer {
    //映射到本地路径的代码意思是
    // 在F:/uploadFiles/user_head/目录下如果有一张aa.png的图片，那么
    // 访问：http://localhost:8081/user_head/aa.png就可以访问到它
    // 静态html 中这样写 <img src="http://localhost:8081/user_head/aa.png">
    // 动态html 中这样写 <img :src="imgUrl"> 把值赋给imgUrl就行，(this.imgUrl="http://localhost:8081/user_head/aa.png")

    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        //映射到本地路径
	    registry.addResourceHandler("/user_head/**").addResourceLocations("file:F:/uploadFiles/user_head/");
	    registry.addResourceHandler("/blog_img/**").addResourceLocations("file:F:/uploadFiles/blog_img/");

        //映射到服务器的保存路
        //记得还要配置Nginx的静态资源代理
        registry.addResourceHandler("/user_head/**").addResourceLocations("file:/www/wwwroot/xuyijie.icu/upload_file/user_head/");
        registry.addResourceHandler("/blog_img/**").addResourceLocations("file:/www/wwwroot/xuyijie.icu/upload_file/blog_img/")
    }
}
```

<font color=#999AAA >其实 “addResourceLocations” 里的路径指图片的真是存储路径，和数据库里存储的图片物理路径的字段的值无关，“数据库”里的路径瞎写都可以，只要对应图片的地址就可以，比如图片存在服务器的 /www/wwwroot/xuyijie.icu/upload_file/user_head/ 目录， “addResourceLocations” 里和数据库里都写 F:// ，这样也可以访问到， addResourceLocations 是解析路径前端的 :src 路径，使图片的物理路径转换为 url 。

<hr style=" border:solid; width:100px; height:1px;" color=#000000 size=1">



# 五、导入数据库


<font color=#999AAA >你创建的数据库右边，有一个管理，点一下，用数据库账号和密码登录进去


![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/宝塔部署9.png)


<font color=#999AAA >左边会显示你创建的数据库，选中它，点上面的“导入”，就可以选择 .sql 文件进行数据导入了

![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/宝塔部署10.png)





<hr style=" border:solid; width:100px; height:1px;" color=#000000 size=1">


# 六、访问自己的项目

在浏览器网址栏输入你 Nginx 那一步配置的 IP 和端口，就可以了，图片也可以上传访问



![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/宝塔部署11.png)


<hr style=" border:solid; width:100px; height:1px;" color=#000000 size=1">


# 总结
<font color=#999AAA >需要的详细后端打包教程和 Linux 服务器上传文件，连接服务器方法和命令，看我的另一篇很详细的，前端打包不需要教程，直接 npm run build 完事儿，还有 XShell 操作服务器和 Xftp 给服务器上传文件不会的，项目启动脚本，链接统一放一下
[Java非Web的Springboot项目打包部署到Linux服务器并运行（Maven、Gradle）](https://blog.csdn.net/qq_48922459/article/details/121718064?spm=1001.2014.3001.5501)
[启动脚本和配置文件下载](https://download.csdn.net/download/qq_48922459/55643280)


<font color=#999AAA >本篇也没有介绍如何从前端上传图片并使用后端处理后讲 base64 转化为 url 存入数据库，需要学习的传送门：[Vue+Springboot上传图片将 Base64 码转换为图片保存在指定文件夹](https://blog.csdn.net/qq_48922459/article/details/122054809)
