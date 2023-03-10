---
title: Linux配置宝塔面板并一键部署网站（免手动安装、免配置环境、可部署多个）
date: 2021-12-09 20:22:34
categories: Linux
tags:
    - Linux
    - 宝塔面板
cover: https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/宝塔面板截图.png
---
# 前言

<font color=#999AAA >你们写项目是为了干什么，写项目的最终目的不就是要部署部署部署！！！上线上线上线！！！那么棒的项目当然要所有人都看见啦，要不然写它用来孤芳自赏吗</font>

<hr style=" border:solid; width:100px; height:1px;" color=#000000 size=1">

<font color=#999AAA >提示：需要有一服务器，虚拟机或者腾讯云购买的都可以，不允许是已经配置了环境的服务器，比如Apache/Nginx/php/MySQL，如果已经安装了这些都要彻底卸载，系统不限（不过听说Windows Server 的服务器用自带的 IIS 比较好，以后讲）


# 一、打包你的前后端项目


<font color=#999AAA >传送门：我的另一篇[Java非Web的Springboot项目打包部署到Linux服务器并运行（Maven、Gradle）](https://blog.csdn.net/qq_48922459/article/details/121718064?spm=1001.2014.3001.5501)

<font color=#999AAA >当然也可以不用看，很简单的，后端 Maven 项目直接在 idea 的 Terminal 控制台运行 `maven clean package` 就可以了，Gradle 在 idea 的右侧 “Gradle” 点开，里面慢慢找，有个“build”，双击。最终都是得到一个 jar 包。

<font color=#999AAA >前端一般用 Vue ，打包方法就是在项目的控制台 `npm run build` 就可以了，会生成一个 dist 文件夹。

==当然，静态网页也可以部署，如果后端没写完，可以只部署前端网页，这篇讲一键建站，所以不讲后端怎么部署和启动，想知道的还是看我上面的那个传送门==

<hr style=" border:solid; width:100px; height:1px;" color=#000000 size=1">

# 二、下载宝塔
<font color=#999AAA >所有系统安装都很傻瓜，Linux 命令行只需一行（如果可以打开图形界面也可以直接去官网下载Linux版），Windows去官网下载 exe 文件一键安装

<font color=#999AAA >系统兼容性推荐 Centos7.x > Debian10 > Ubuntu 20.04 > Cenots8.x > Ubuntu 18.04 > 其它系统

Centos安装脚本 `yum install -y wget && wget -O install.sh http://download.bt.cn/install/install_6.0.sh && sh install.sh`
Ubuntu/Deepin安装脚本 `wget -O install.sh http://download.bt.cn/install/install-ubuntu_6.0.sh && sudo bash install.sh`
Debian安装脚本 `wget -O install.sh http://download.bt.cn/install/install-ubuntu_6.0.sh && bash install.sh`
Fedora安装脚本 `wget -O install.sh http://download.bt.cn/install/install_6.0.sh && bash install.sh`

<hr style=" border:solid; width:100px; height:1px;" color=#000000 size=1">

#  三、登录宝塔面板


<font color=#999AAA >安装成功后在随便一台电脑的浏览器输入 bt.cn 进入官网，安装 ==堡塔SSH终端==连接到你的服务器，也可以不下载，直接在电脑浏览器里输入 ==服务器IP:8888== 就可以访问到服务器里的宝塔，记得开放服务器的防火墙


<font color=#999AAA >然后刚进去就提示你安装套件，强烈推荐！！！直接安装 LNMP 就行了，记得选 “编译安装”，差不多半小时，就可以了。

![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/宝塔0.png)

<font color=#999AAA >一顿简单的操作，最终你的宝塔界面是这样的，点击左边 “网站”

![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/宝塔面板截图.png)

<font color=#999AAA >根据我的标注操作，域名那里如果没有域名，就写自己的服务器IP，端口可以指定，但是不允许是8080，记得开放服务器防火墙，数据库可以选择创建或者以后再创建，提交
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/宝塔1.png

<font color=#999AAA >我以前已经部署了两个了，两个网站写不同的端口就行了，点开网站根目录，初始有两张页面，我们可以把 index.html 和 404.html 删了，因为要放我们的 Vue 项目

![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/宝塔2.png)
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/宝塔3.png)


<font color=#999AAA >直接把打包好的 dist 文件夹里面的东西，上传到这里就行了，然后就可以访问了，神奇吧

![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/宝塔4.png)

<hr style=" border:solid; width:100px; height:1px;" color=#000000 size=1">


#  三、访问自己的网站

<font color=#999AAA >浏览器输入自己配置的域名，我的是 175.24.228.202:8081  ，完成
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/宝塔5.png)




<hr style=" border:solid; width:100px; height:1px;" color=#000000 size=1">

# 总结
<font color=#999AAA >暂时没有总结吧，总之觉得宝塔还不错，文件、网站、数据库、FTP、VPN 这些都可以在上面管理，告别命令行，告别繁琐的环境配置
