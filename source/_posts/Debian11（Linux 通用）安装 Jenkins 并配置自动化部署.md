---
title: Debian11（Linux 通用）安装 Jenkins 并配置自动化部署
date: 2022-10-28 18:15:19
categories: Linux
tags:
    - Deepin
    - Linux
    - Ubuntu
    - Debian
    - Jenkins
    - 运维
    - 自动化
cover: https://img-blog.csdnimg.cn/c2b2f110b7dc4f06a816afe49b3d9622.png
---
# 前言
> Jenkins是基于Java开发的一种持续集成工具，用于监控持续重复的工作，可用于自动化各种任务，如构建，测试和部署软件。Jenkins可以通过 apt 和 yum 安装、Docker安装，也可以下载 war 包允许在拥有 JDK 环境的任何机器

---


# 一、安装 Jenkins
`注意，安装 Jenkins 需要 JDK 环境，如果没有安装，可以直接 apt install default-jdk 或者 yum install java 这样安装`

`注意，配置自动化部署需要 maven 环境，如果没有安装，可以直接 apt install maven 或者 yum install maven 这样安装`

`注意，配置自动化部署需要 git 环境，如果没有安装，可以直接 apt install git 或者 yum install git 这样安装，这个 git 的版本不重要，也可以不安装，Jenkins安装好以后让Jenkins自动安装`

## apt/yum安装（推荐）
> 这是 apt 的安装方式

```bash
# 下载Jenkins公钥
curl -fsSL https://pkg.jenkins.io/debian/jenkins.io.key | sudo tee \
    /usr/share/keyrings/jenkins-keyring.asc > /dev/null
# 把Jenkins安装包添加到apt仓库
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
    https://pkg.jenkins.io/debian binary/ | sudo tee \
    /etc/apt/sources.list.d/jenkins.list > /dev/null
# 更新仓库
apt update
apt install jenkins
# 查看Jenkins运行状态
systemctl status jenkins.service
```
> 这是 yum 的安装方式

```bash
# 下载Jenkins仓库
sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
# 安装ca-certificates并导入公钥
yum -y install ca-certificates
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
yum -y install jenkins
```

## Docker安装

```bash
docker pull jenkinsci/blueocean
docker run \
  -u root \
  -d \
  -p 8080:8080 \
  -p 50000:50000 \
  -v jenkins-data:/var/jenkins_home \
  -v /var/run/docker.sock:/var/run/docker.sock \
  jenkinsci/blueocean
```

## War包安装
[点击下载Jenkins的war包](http://mirrors.jenkins.io/war-stable/latest/jenkins.war)

直接java -jar 启动就可以

```bash
# 临时启动
java -jar jenkins.war
# 启动并保持后台运行
nohup java -jar jenkins.war &
```

---

# 二、配置文件位置

```bash
vim /lib/systemd/system/jenkins.service
```

可以修改一些常用配置，只是示例，这里不用改

```bash
User=jenkins
Group=jenkins
Environment="JENKINS_HOME=/opt/jenkins"
WorkingDirectory=/opt/jenkins
Environment="JENKINS_PORT=8080"
```

---

# 三、登录Jenkins控制面板

> 开放防火墙 8080 端口

```bash
# Debian/Ubuntu ufw
ufw allow 8080
ufw reload
# Debian/Ubuntu iptables（这个叼毛防火墙好麻烦，我没用过，不知道是不是这样）
iptables -A INPUT -p tcp --dport 8080 -j ACCEPT
iptables-restore
# CentOS
firewall-cmd --zone=public --add-port=8080/tcp --permanent
firewall-cmd --reload
```

> 浏览器访问服务器IP:8080

![在这里插入图片描述](https://img-blog.csdnimg.cn/1f872289ab2e43228b05ae3758c6d572.png)

> 可以看到首次进入需要解锁，它说密码已经写入到我们的安装服务器，路径是 `/var/lib/jenkins/secrets/initialAdminPassword`，我们 `cat` 一下这个路径，得到密码，粘贴到网页，点击继续

![在这里插入图片描述](https://img-blog.csdnimg.cn/cf75372bf02442d6bc07ebfe67ad74c6.png)
> 我们选择安装推荐的插件就行

![在这里插入图片描述](https://img-blog.csdnimg.cn/d0d94d95b92d4ad58a523e2a02c5f614.png)
> 正在安装，等待安装完成

![在这里插入图片描述](https://img-blog.csdnimg.cn/d59b1a7edea646988f8cb95099058c74.png)
> 创建用户

![在这里插入图片描述](https://img-blog.csdnimg.cn/ff0579645bbe4832b8a35bc1616d4939.png)
> 就是控制面板的地址，保存继续就OK了！！！

![在这里插入图片描述](https://img-blog.csdnimg.cn/29856b25e1eb484e8e2e289703acf39c.png)


---

# 四、配置自动化部署
> 我将给 Jenkins 配置 JDK环境、Maven环境、Git地址、项目运行服务器IP来实现对项目的监控、自动部署

## 1、配置环境
> 查看 java_home：1、如果你是通过解压压缩包安装的，直接`echo $JAVA_HOME`。2、如果是通过 apt 和 yum 安装的，`update-alternatives --config java`查看，`/usr/lib/jvm/java-17-openjdk-amd64`就是 java_home 的位置
> 查看 Maven_home：mvn -v

```bash
root@debian:/usr/lib/jvm/java-1.17.0-openjdk-amd64# echo $JAVA_HOME

root@debian:/usr/lib/jvm/java-1.17.0-openjdk-amd64# update-alternatives --config java
链接组 java (提供 /usr/bin/java)中只有一个候选项：/usr/lib/jvm/java-17-openjdk-amd64/bin/java
无需配置。
root@debian:/usr/lib/jvm/java-1.17.0-openjdk-amd64# mvn -v
Apache Maven 3.6.3
Maven home: /usr/share/maven
Java version: 17.0.4, vendor: Debian, runtime: /usr/lib/jvm/java-17-openjdk-amd64
Default locale: zh_CN, platform encoding: UTF-8
OS name: "linux", version: "5.10.0-18-amd64", arch: "amd64", family: "unix"

```

> 获取完成，开始配置，把 Install automatically 钩打掉就能配置本地环境了

![在这里插入图片描述](https://img-blog.csdnimg.cn/9be62dc5ed9243bdae52a23f55bd0531.png)


![在这里插入图片描述](https://img-blog.csdnimg.cn/db7e4865e0be4b55a1d6c2b29e7d46c1.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/533ba375c1c949c398daf7b146042fb4.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/15de9dfa01fc4555a78d495d7f915c1a.png)

---

## 2、配置自动化部署

> 配置好环境保存，新建Item

![在这里插入图片描述](https://img-blog.csdnimg.cn/b09e02aa0085498a94973d1c02c63478.png)
> 名字随便，然后选择`Freestyle project`

![在这里插入图片描述](https://img-blog.csdnimg.cn/ea9e6344b87a49fd86408629836794ef.png)

> 勾选为 Github项目，输入仓库地址

![在这里插入图片描述](https://img-blog.csdnimg.cn/3d16ef9047a34a299a2cc76dca26904a.png)
> 配置项目的 Git 仓库连接，`Credentials`自己添加一个，填 github 的登录账号密码

![在这里插入图片描述](https://img-blog.csdnimg.cn/cb11f579c1174ebbbb457dadf0749ca7.png)

> 构建触发器身份令牌随便填
> 如下图：当前项目的回调地址为：`http://localhost:8080/me/my-views/view/all/job/test/build?token=test`
只要执行这个地址（在浏览器上访问改地址），该项目就会发起一次构建项目，即拉取代码打包部署操作

![在这里插入图片描述](https://img-blog.csdnimg.cn/6657740438e44aceaa6205609f013404.png)
> 配置构建命令：打包，cd 到 jar 包位置，启动，别问 BUILD_ID 是什么意思，我也不知道，就是这样写的

![在这里插入图片描述](https://img-blog.csdnimg.cn/27da6ea21d104ac5aff5ebe683f876ac.png)

## 远程部署（可选）

> 如果想打包到其他服务器启动，要安装新插件

![在这里插入图片描述](https://img-blog.csdnimg.cn/aeb4f6aa6d9c482ea09fa6a2b32fc283.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/bc91da9a186248fc8ff284fb925d1744.png)
> 页面左边还有一个 Download program，点进去，拉到最下面，打勾“完成后重启”
> 如果没有，可以手动重启 `systemctl restart jenkins`

![在这里插入图片描述](https://img-blog.csdnimg.cn/c2b2f110b7dc4f06a816afe49b3d9622.png)

> 配置 `Send build over SSH` 因为我这里只有一台服务器，没有配置 ssh，我网上找了一张老图

![在这里插入图片描述](https://img-blog.csdnimg.cn/8a2f665abb0c47deb94b9cf0d8a7ce87.png)




> 在GitHub服务器上的指定项目里面配置上文中提到的回调地址

![在这里插入图片描述](https://img-blog.csdnimg.cn/5001ad0c003b482e89b9c3ade0fe5a53.png)


---
# 总结
> 配置好以后，直接向配置的 Git 仓库分支推送代码，Jenkins会自动开始构建，这里可以看到构建历史

![在这里插入图片描述](https://img-blog.csdnimg.cn/4b3ef98df783499eac9c49cdb20b0dac.png)


除了 Jenkins，还有一款免费的叫 JPom，也很不错，推荐一下，有兴趣的可以研究一下
