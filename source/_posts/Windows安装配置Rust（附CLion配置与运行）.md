---
title: Windows安装配置Rust（附CLion配置与运行）
date: 2024-04-18 19:18:01
categories:
  - Rust
tags:
  - SpringBoot
  - logback
cover: https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/彩色日志.png
---

# 前言

> 本文以 windows 安装为例，配置编译器为 minGW，使用 clion运行，可以不用下载 vs 和众多依赖

---

# 一、下载

> 点击进入 [rust官方网站](https://www.rust-lang.org/zh-CN/tools/install) 进行下载，我们选择64位的下载

![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/Rust下载界面.png)

---

# 二、安装

> 如果你想`修改rust的安装位置`(默认C盘)，下载完成后先`不要`打开，我们要先配置环境变量，`CARGO_HOME`，值是你想安装的位置

![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/Cargo环境变量.png)




> 右键以管理员身份运行，会出现下面的弹窗，我们选 3，回车，这样可以`免安装visual studio`这种重量级软件


![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/安装Cargo1.png)

> 接下来是这样，安装位置就是你刚才配置的环境变量位置

![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/安装Cargo2.png)
> 如果你已经安装 MSVC ，那么安装过程会非常的简单，输入 1 并回车，直接进入第二步，但是相信大多数同学都没有安装，一是要下载vs，很麻烦，二是使用`minGW`可以跨平台，所以我们`选择 2` ，回车，再输入`x86_64-pc-windows-gnu`，回车


![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/安装Cargo3.png)
> 然后会提示我们选择工具版本，我们输入`stable`回车，然后是 which tolls and data to install 提示安装内容，默认就是  default ，`直接回车`就行，然后输入`Y`回车

![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/安装Cargo4.png)
> 发现又回到了刚打开时的解面，这时候，我们的 default host trip 就变成了我们刚才设置的`gnu`，也就是minGW，输入`1`,回车，等待下载完成

![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/安装Cargo5.png)
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/安装Cargo6.png)


> 可以查看rust版本，出现版本号妥妥的安装成功

![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/验证Cargo.png)

---

# 三、配置标准库！！！

> ==还有一步很重要的，我发现大部分教程都没写==，执行以下下面的命令，否则 rustup 标准库缺失，导致程序无法运行：`rustup component add rust-src`


![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/配置rustup1.png)

> 配置环境变量，`RUSTUP_HOME`

![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/配置rustup2.png)


---


# 四、使用 CLion 运行 rust

## 1、新建rust项目

> new project 选择 rust 项目，第一次新建会提示你下载 rust 插件，下载完成后新建界面就长这样，location是项目位置，下面的都是clion自动识别的，如果你没有执行上面安装rustup component add rust-src这个命令，这里就会识别不出来

![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/新建rust项目1.png)

## 2、配置运行环境

> create，新建完成，首次使用 clion，他会提示你创建运行环境，我们添加一个 `MinGW`就可以了，clion里面内置的就有，无需自己出去下载

![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/新建rust项目2.png)

> 如果没有弹出这个弹窗，你也没有配置`MinGW`，没关系，你去`setting的build下面的Toolchains`也可以打开这个界面的

![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/新建rust项目3.png)
## 3、运行

> 配置好编译器后，等待项目构建完成，项目初始结构就长这样

![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/编写rust1.png)
> 运行一下，完美

![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/编写rust2.png)
