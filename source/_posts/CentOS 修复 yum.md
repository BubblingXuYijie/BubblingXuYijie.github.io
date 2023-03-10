---
title: CentOS 修复 yum
date: 2022-01-06 14:52:43
categories: Linux
tags:
    - Linux
    - CentOS
    - yum
cover: https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/LinuxLogo.jpg
---
# 前言

<font color=#999AAA >在配置 yum 以后，也许某一天这个源失效了，你也无法使用 wget 下载其他的源，好吧如果你下载了其他的源，但是也可能会遇到 yum 无法刷新缓存或者一直卡在fastestmirror和下载界面的情况，当你如此绝望的要重装 yum 甚至重装系统的时候，那么你只需要 `3` 步就可以挽救你的系统。</font>

<hr style=" border:solid; width:100px; height:1px;" color=#000000 size=1">


# 一、删除原来的配置

```bash
rm -rf /etc/yum.repos.d/*
```
<hr style=" border:solid; width:100px; height:1px;" color=#000000 size=1">

# 二、官方途径

```bash
rpm -Uvh --force http://mirror.centos.org/centos-7/7/os/x86_64/Packages/centos-release-7-9.2009.0.el7.centos.x86_64.rpm
```

注意后面的 `http://mirror.centos.org/centos-7/7/os/x86_64/Packages/centos-release-7-9.2009.0.el7.centos.x86_64.rpm` 官方会删除旧版，只留下新版，所以先打开这个网址看看， 404 的话就访问这个网址`http://mirror.centos.org/centos-7/7/os/`，寻找`centos-release-7-9.2009.0.el7.centos.x86_64.rpm`这种样式的复制链接再运行命令。
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/CentOSDownload.png)
<hr style=" border:solid; width:100px; height:1px;" color=#000000 size=1">

#  三、重新生成缓存

```bash
yum clean all		# 清空缓存 
yum makecache		# 生成缓存
```


<hr style=" border:solid; width:100px; height:1px;" color=#000000 size=1">

# 总结
<font color=#999AAA >完
