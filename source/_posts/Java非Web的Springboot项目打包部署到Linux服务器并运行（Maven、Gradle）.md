---
title: Java非Web的Springboot项目打包部署到Linux服务器并运行（Maven、Gradle）
date: 2021-12-04 17:58:29
categories: Linux
cover: https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/springbootLogo.jpeg
---
# 前言

<font color=#999AAA >直接开始吧，非Web项目部署很简单的，没有繁杂的配置</font>

<font color=#999AAA >有些小伙伴说非Web的项目运行一下就自己停止了，我还写了一篇很简单的设置定时任务，就是每隔5秒运行一次，永不停止这样，传送门：[Java实现非Web项目的Springboot定时任务（每3秒自动执行一次）](https://blog.csdn.net/qq_48922459/article/details/121687993?spm=1001.2014.3001.5501)</font>

<hr style=" border:solid; width:100px; height:1px;" color=#000000 size=1">




# 一、修改pom.xml配置文件


<font color=#999AAA >我以Maven为例

<font color=#999AAA >我的项目在本地启动以后是这样的，等下部署后在服务器启动也是这个样子的，先停止它。

![在这里插入图片描述](https://img-blog.csdnimg.cn/c3903cd5384b4729b39ab1484bdcc205.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA57mB5Y2O5bC95aS05ruh5piv5q6H,size_20,color_FFFFFF,t_70,g_se,x_16)


<font color=#999AAA >打开pom.xml，对比一下你的 `<build>` 里面是不是缺少下方的代码，自行添加缺少的配置。注意！如果你的 `<build>` 里面 spring-boot-maven-plugin 的 configuration 里面有 ==lombok== 的配置，一定要==删掉==，不然可能会打包失败，删掉lombok的配置不会影响lombok的使用。`<mainClass>`标签里面是自己项目的主启动类路径，自己换一下。


```xml
	<build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <excludes>
                        <!--打包部署需要的配置，开启true-->
                        <includeSystemScope>true</includeSystemScope>
                    </excludes>
                </configuration>
            </plugin>

            <!--打包部署需要的配置-->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <configuration>
                    <skipTests>true</skipTests>
                </configuration>
            </plugin>
            <!--解决打包部署报错no main manifest attribute, in 。。。jar-->
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <mainClass>com.zjsos.pingyangparkinglot.PingyangParkinglotApplication</mainClass>   
                   	<!--第三方包需要这个配置才能打包进去-->
					<includeSystemScope>true</includeSystemScope>	
                </configuration>
                <executions>
                    <execution>
                        <goals>
                            <goal>repackage</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>

            <plugin>
                <!--热部署配置-->
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <!--fork：如果没有该项配置，整个devtools不会起效果-->
                    <fork>true</fork>
                </configuration>
            </plugin>
        </plugins>
    </build>
```


<hr style=" border:solid; width:100px; height:1px;" color=#000000 size=1">

# 二、开始打包

<font color=#999AAA >首先，点击idea下方的“Terminal”

![在这里插入图片描述](https://img-blog.csdnimg.cn/5ec1a80182864bd29a12aac0de14da39.png)

<font color=#999AAA >然后在里面输入“`mvn clean`”，回车

![在这里插入图片描述](https://img-blog.csdnimg.cn/670f8ec7739f409892ef733dcce0eece.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA57mB5Y2O5bC95aS05ruh5piv5q6H,size_20,color_FFFFFF,t_70,g_se,x_16)

<font color=#999AAA >等待它完成后，再输入“`mvn package`”，回车，等待出现到这一步，打包就完成啦，jar包路径在项目根目录的target文件夹里

![在这里插入图片描述](https://img-blog.csdnimg.cn/c5d147f966c743cfb63519e3f9b2d3c1.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA57mB5Y2O5bC95aS05ruh5piv5q6H,size_20,color_FFFFFF,t_70,g_se,x_16)

<font color=#999AAA >打开 target ，里面有以你的项目名命名的jar包

![在这里插入图片描述](https://img-blog.csdnimg.cn/99d8b126ba0a4fca84506b8f52bbd29a.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA57mB5Y2O5bC95aS05ruh5piv5q6H,size_16,color_FFFFFF,t_70,g_se,x_16)

<font color=#999AAA >部署到服务器之前可以先测试一下jar包是否正常，在jar包所在的文件夹路径栏输入cmd，回车，会打开命令行

![在这里插入图片描述](https://img-blog.csdnimg.cn/347bb37ad7d64efcb91eb75135d83949.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA57mB5Y2O5bC95aS05ruh5piv5q6H,size_19,color_FFFFFF,t_70,g_se,x_16)


<font color=#999AAA >在命令行里输入 `java -jar 你的jar包名.jar` ，回车，即可运行


![在这里插入图片描述](https://img-blog.csdnimg.cn/6a946529cf60417c8e42e7e83d3f1628.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA57mB5Y2O5bC95aS05ruh5piv5q6H,size_20,color_FFFFFF,t_70,g_se,x_16)

<font color=#999AAA >运行正常，我们把它部署到服务器

<hr style=" border:solid; width:100px; height:1px;" color=#000000 size=1">

# 三、部署到服务器


<font color=#999AAA >服务器系统以没有界面的 Centos7 为例，连接工具使用的是 Xshell ，打开 Xshell ，左上角新建，“主机”输入要连接的服务器IP，端口如果没有特别改动的话一般是 22 或者 122

![在这里插入图片描述](https://img-blog.csdnimg.cn/bd1f54c8c72b403aa4d8b1078447c746.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA57mB5Y2O5bC95aS05ruh5piv5q6H,size_20,color_FFFFFF,t_70,g_se,x_16)

<font color=#999AAA >输入完 IP 和端口以后，点击左边“用户身份验证”，输入账号密码，点连接


![在这里插入图片描述](https://img-blog.csdnimg.cn/5ea2b3957fcb4d5f836c9049ba49d6f9.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA57mB5Y2O5bC95aS05ruh5piv5q6H,size_16,color_FFFFFF,t_70,g_se,x_16)

<font color=#999AAA >出现 Welcome to xxx 连接成功

![在这里插入图片描述](https://img-blog.csdnimg.cn/24db33e4d9b34efe8f76d7df14e3ff95.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA57mB5Y2O5bC95aS05ruh5piv5q6H,size_12,color_FFFFFF,t_70,g_se,x_16)

<font color=#999AAA >把项目部署在我们挂载的主目录，挂载就不讲了，你们购买的服务器都是挂载好的，弄清楚是哪个文件夹就可以了，我的是/data，进入到/data，创建项目文件夹，进入文件夹

![在这里插入图片描述](https://img-blog.csdnimg.cn/d8cc776653b444d7aedde6371782ac03.png)

<font color=#999AAA >然后点击Xshell上面这个绿色的，是开启本机和服务器图形界面的文件传输框Xftp，左边在本机找到jar包，拖到右边服务器

![在这里插入图片描述](https://img-blog.csdnimg.cn/fd1a0ed923e64a29b2890ad80ec0d762.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/ae7a047cb7cb4cba9d40a1412cfd66be.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA57mB5Y2O5bC95aS05ruh5piv5q6H,size_20,color_FFFFFF,t_70,g_se,x_16)

<font color=#999AAA >然后返回到 Xshell ，运行jar包，但是这种运行方式只是暂时的，离开这个界面就会停止，下面的方法可以让项目一直运行

![在这里插入图片描述](https://img-blog.csdnimg.cn/60723dbe1d3a4471bc7b1a95cac56025.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA57mB5Y2O5bC95aS05ruh5piv5q6H,size_20,color_FFFFFF,t_70,g_se,x_16)

<font color=#999AAA >首先下载我给的文件，这是jar在linux上启动的通用脚本，不需要积分
[启动脚本和配置文件下载](https://download.csdn.net/download/qq_48922459/55643280)
<font color=#999AAA >下载完成后把解压出来的文件夹里的文件拖到服务器jar包所在的文件夹

![在这里插入图片描述](https://img-blog.csdnimg.cn/9f7429df33c648cab0284de905c0d34c.png)

<font color=#999AAA >输入 `ls` 查看文件，然后输入 `sh build.sh` 来运行这个文件，会显示构建成功，这时会发现多了一个autoScript文件夹，`cd autoScript`，里面有几个sh文件，`sh startup.sh` ，项目就会启动了，并且永远不会停止，除非你运行 `sh shutdown.sh`。OK，完成。放心的退出你的服务器吧。

![在这里插入图片描述](https://img-blog.csdnimg.cn/e35c42596c58477db98839a40eb57e1c.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA57mB5Y2O5bC95aS05ruh5piv5q6H,size_20,color_FFFFFF,t_70,g_se,x_16)
<hr style=" border:solid; width:100px; height:1px;" color=#000000 size=1">

# 总结
<font color=#999AAA >本编以Maven为例打包，但是打完包以后，部署到服务器的操作都是一样的，启动脚本里面的 indolent.properties 配置文件可以配置 nacos 上的配置文件，个人的小项目暂时用不到，后面会写部署Springboot的Web项目，这个环境的配置就比较复杂了，但是我目前用的是一键部署的神器，宝塔面板，不用安装和配置环境，传送门：[记Linux使用宝塔部署Vue+Springboot前后端分离项目](https://blog.csdn.net/qq_48922459/article/details/121901441?spm=1001.2014.3001.5501)。
