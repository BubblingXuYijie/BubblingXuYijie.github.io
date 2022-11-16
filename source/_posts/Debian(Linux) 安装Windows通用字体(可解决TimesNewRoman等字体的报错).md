---
title: Debian(Linux) 安装Windows通用字体(可解决TimesNewRoman等字体的报错)
date: 2022-05-09 17:43:53
categories: Linux
tags:
    - Debian
    - Linux
    - Ubuntu
cover: /img/LinuxLogo.jpg
---
# 前言
最近写了个小玩意儿，PDF转Word，体验很棒，图片和插画都能识别出来正确转换，可是部署到线上以后，转换会有Arial和TImesNewRoman字体的报错

原因就是Linux的字体和Windows的不太一样，毕竟PDF都是WIndows上面保存下来的。



---

# 一、直接操作

```bash
sudo apt install ttf-mscorefonts-installer # 安装
#这里刷新两遍缓存，保险
sudo fc-cache -f -v
sudo fc-cache
#测试一下TimesNewRoman这个字体有没有安装成功
fc-match Times
```
大功告成
![在这里插入图片描述](https://img-blog.csdnimg.cn/b24596b2bceb489a931a2d09ac0304a0.png)
然后一定要重启报错的项目



---

# 总结
完
