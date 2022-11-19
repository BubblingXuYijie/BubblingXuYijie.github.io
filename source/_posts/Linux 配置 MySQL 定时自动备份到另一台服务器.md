---
title: Linux 配置 MySQL 定时自动备份到另一台服务器
date: 2022-01-05 13:58:04
categories: Linux
tags:
    - Linux
    - MySQL
    - Shell脚本
cover: https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/MysqlLogo.jpg
---
# 前言

<font color=#999AAA >此方案可使一台服务器上的 MySQL 中的所有数据库每天 0 点自动转储为 .sql 文件，然后将文件同步到另一台服务器上，可以作为一个简单的数据容灾。</font>

<hr style=" border:solid; width:100px; height:1px;" color=#000000 size=1">



# 一、配置服务器 ssh


<font color=#999AAA >从一台服务器同步文件到另一台服务器，需要使两台服务器之间建立 ssh 连接，不然每次执行备份的时候你还要半夜爬起来去输入服务器密码。

```bash
ssh-keygen -t rsa
```
<font color=#999AAA >运行这句，一直敲击回车，会在 `/root/.ssh` 目录下生成两个文件，我使用的是 xftp 查看的文件，这个 .ssh 文件夹是`隐藏文件夹`，所以 xftp 下面看不到，你要手动输入地址 /root/.ssh 就可以进去了，把 `id_rsa.pub` 里的内容全选复制。
![在这里插入图片描述](https://img-blog.csdnimg.cn/7cc7c8c05a2f44709b63d08ba532e302.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA57mB5Y2O5bC95aS05ruh5piv5q6H,size_12,color_FFFFFF,t_70,g_se,x_16)
![在这里插入图片描述](https://img-blog.csdnimg.cn/888bb1f1266f4ea2b559b87ca95a93cc.png)


<font color=#999AAA >来到目标服务器，也是进入 `/root/.ssh` 文件夹，把刚刚复制的内容粘贴进 `authorized_keys` 的尾部（authorized_keys 可能没有，自己新建，里面可以有多个 key，互不影响，换一行粘贴就行了）

![在这里插入图片描述](https://img-blog.csdnimg.cn/4ad1a3757b8f4d5a88710cccf5d7fac3.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA57mB5Y2O5bC95aS05ruh5piv5q6H,size_19,color_FFFFFF,t_70,g_se,x_16)
![在这里插入图片描述](https://img-blog.csdnimg.cn/2dd8151717dc428891e391eb83a4c8c2.png)


<font color=#999AAA >保存，这样我们两台服务器的 ssh 连接就建立好了。
<hr style=" border:solid; width:100px; height:1px;" color=#000000 size=1">


# 二、编写自动备份 sh 文件


<font color=#999AAA >在数据库所在的服务器新建一个 sh 文件，放在哪里、怎样命名都随意（`此处注意！！！不要使用 xftp 右键新建文件来编写 sh 文件，那样编码会出问题，比如新建的文件名后面会多出一个问号“?”，使用命令行vim或vi来新建`）

```bash
mkdir mysqlAutoBackupTo24
cd mysqlAutoBackupTo24
mkdir backup
vim AutoBackup.sh
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/c7a9227f880d422a84036059e2780ad4.png)
<font color=#999AAA >我建立的文件夹是这样的

![在这里插入图片描述](https://img-blog.csdnimg.cn/7fe3aba1924d4d3e88d4399145407839.png)

<font color=#999AAA >AutoBackup.sh 里面的内容

```powershell
#下面生成的sql在本服务器存放的文件夹，就是我上面建立的
BACKUP=/data/mysqlAutoBackupTo24/backup
#当前时间，用来命名sql文件
DATETIME=$(date +%Y-%m-%d)
echo "===备份开始==="
echo "备份文件存放于${BACKUP}/$DATABASE-$DATETIME.sql"
#生成sql文件，命名
DATABASE=dbBackup
echo $DATABASE-$DATETIME
#mysqldump -h localhost -u${DB_USER} -p${DB_PW} --all-databases > ${BACKUP}/$DATABASE-$DATETIME.sql
mysqldump -h 11.11.111.28 -uroot -proot --databases dba dbb > ${BACKUP}/$DATABASE-$DATETIME.sql
echo "===导出成功，开始传输==="
#将sql文件从服务器28备份到服务器24自己建立的文件夹/data/mysqlAutoBackupFrom28下面
scp -P 122 $DATABASE-$DATETIME.sql root@11.11.11.24:/data/mysqlAutoBackupFrom28
#删除备份目录
#rm -rf ${BACKUP}/$DATETIME
#删除7天前备份的数据，自行更改
#find $BACKUP -mtime +7 -name "*.sql" -exec rm -rf {} \;
echo "===数据库备份到服务器成功==="

```

<font color=#999AAA >`mysqldump` 的 -h 后面写所在服务器的 IP，也就是24，不要写 localhost，因为如果是离线安装的 mysql 可能没有 mysqld.socket ，导致连接失败。

<font color=#999AAA >`scp -P 122` 是指定 ssh 端口，不指定默认为 22，root 是目标服务器24 的用户名

<font color=#999AAA >`scp` 的 `--databases dba dbb` 的意思是指定备份 dba 和 dbb 这个两个数据库，指定多个数据库要加 --databases ，数据库用空格隔开，上面一句注释掉的是 `--all-databases` ，意思是备份全部数据库，但是我的报错，不知道为什么。你们可以试一下。


下面我们运行一下这个 sh 看看效果，cd 到你的 sh 存放的文件夹 `sh AutoBackup.sh`，首次进行 ssh 连接要输入一个 `yes` 回车，然后去目标服务器 24 查看，sql 文件已经同步过去。

![在这里插入图片描述](https://img-blog.csdnimg.cn/6ded890c67f94762a535dd6656502a8d.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/f0a03101fe4f4443a2a417a15a4b300e.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA57mB5Y2O5bC95aS05ruh5piv5q6H,size_15,color_FFFFFF,t_70,g_se,x_16)


<hr style=" border:solid; width:100px; height:1px;" color=#000000 size=1">


# 三、设置定时自动执行

<font color=#999AAA >上面的测试没有问题，下面我们设定一个每天 0 点自动执行 sh 脚本，就可以失效每天的自动同步。

<font color=#999AAA >首先赋予要执行的 shell 脚本权限，给高一点，不然没法自动执行

```bash
chmod 777 /data/mysqlAutoBackupTo24/AutoBackup.sh
```

<font color=#999AAA >输入下面语句，vim 会打开一个文件
```bash
crontab -e
```
<font color=#999AAA >里面这样写，保存，前面的 `02 00 * * *` 是 cron 表达式，代表每天 00:02 执行 `/data/mysqlAutoBackupTo24/AutoBackup.sh`，之所以设置 00:02 是因为避免服务器在 0 点的时候有其他数据同步任务，所以晚一点。cron 表达式的语法你们可以学一下。

```bash
02 00 * * * sh /data/mysqlAutoBackupTo24/AutoBackup.sh
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/a8db4aa12a004fe0989d811cfe2eed1f.png)

<font color=#999AAA >保险起见再刷新一下配置
```bash
service crond reload
```

<hr style=" border:solid; width:100px; height:1px;" color=#000000 size=1">

# 总结
<font color=#999AAA >感谢https://blog.csdn.net/qq_33966519/article/details/103761673这篇文章，Windows 的 MySQL 备份要编写一个 bat 文件，在系统自带的计划任务里设置定时，后面研究研究再写吧。
