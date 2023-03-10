---
title: Linux系统一键安装Docker（并更改Docker的容器默认存储位置）
date: 2021-12-27 17:32:54
categories: Linux
tags:
    - Linux
    - Docker
    - apt
cover: https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/DockerLogo.jpg
---
# 前言

<font color=#999AAA >Linux系统Docker的安装非常简单，官方的一键安装命令，无需配置任何东西，如果你的服务器无法联网就只能用麻烦的离线安装了。</font>

<font color=#999AAA >就是如果你的服务器的磁盘分了不同区的话，要更改一下 Docker 的容器和镜像的默认存储位置，否则的话默认会安装在`/var/lib/docker`目录下，日志也会在这个目录生成，慢慢把系统盘空间占满。</font>

<hr style=" border:solid; width:100px; height:1px;" color=#000000 size=1">


# 一、一键安装命令


##  Ubuntu、Debian、UOS、Deepin、CentOS都一样

```bash
curl -sSL https://get.daocloud.io/docker | sh
```

<font color=#999AAA >等待安装完成后，运行`docker ps`，这样就算是安装好了

![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/安装Docker0.png)
<font color=#999AAA >启动服务

```bash
systemctl start docker.service
```

<font color=#999AAA >开机自启

```bash
sudo systemctl enable docker
```


<hr style=" border:solid; width:100px; height:1px;" color=#000000 size=1">

# 二、更改默认存储路径



<font color=#999AAA >命令行输入 `vi /etc/docker/daemon.json`，在这个文件里输入以下内容，/data/docker 是要修改的存储路径，你们看自己想修改到哪里。

<font color=#999AAA >vi 的操作和 vim 一样，进去以后按一下 “a”就可以输入了，然后保存是先按一下“esc”，再按住“shift + z + z”就可以了

```bash
{
    "graph":"/data/docker"
}
```

![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/安装Docker1.png)

<font color=#999AAA >然后重启一下 Docker ，命令行输入 `systemctl restart docker.service`


<font color=#999AAA >然后命令行输入`docker info`，看我鼠标拉白的那个路径，我没有改，所以还是原来的路径

![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/安装Docker2.png)







<hr style=" border:solid; width:100px; height:1px;" color=#000000 size=1">

# 总结
<font color=#999AAA></font>
