---
title: Nginx 配置 SSL 证书（网站变为 HTTPS）
date: 2021-12-31 10:24:19
categories: Python
tags:
    - Nginx
    - SSL
    - HTTPS
    - Linux
cover: /img/NginxLogo.png
---
# 前言

<font color=#999AAA >一键建站工具例如 宝塔、wordpress 都有一键配置 https 的选项，虽然我写了这篇文章，但我还是推荐使用建站软件嘿嘿，没有建站软件的小伙伴门只能自己配了，但也是非常简单的。</font>

<hr style=" border:solid; width:100px; height:1px;" color=#000000 size=1">



# 一、把 SSL 证书的上传到服务器


<font color=#999AAA >把ssl证书的pem和key传到服务器的nginx的根目录的cert（这个文件夹自己创建，可以不叫这个名字），这两文件 pem 和 key 传上来就可以了。

![在这里插入图片描述](https://img-blog.csdnimg.cn/9b7048dacf34491991e26f90fa61c9d8.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/7dc38fee577a43ca9dce17c4dd4e8c93.png)

# 二、修改 nginx.conf

<font color=#999AAA >然后修改nginx.conf，添加（路径和端口和 ip 自己修改）：

```xml
server {
	listen   443;
	server_name  111.111.111.111; 	
	ssl on;
	# 下面两个 pem 和 key 换成你自己证书的
	ssl_certificate /data/nginx/cert/6722197_godata.dongtou.gov.cn.pem;
	ssl_certificate_key /data/nginx/cert/6722197_godata.dongtou.gov.cn.key;
	ssl_session_timeout 5m;
	ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
	ssl_protocols TLSv1 TLSv1.1 TLSv1.2;	
	ssl_prefer_server_ciphers on;
```




<hr style=" border:solid; width:100px; height:1px;" color=#000000 size=1">

# 三、重启 Nginx
<font color=#999AAA >然后保存，重启 nginx 就可以了
