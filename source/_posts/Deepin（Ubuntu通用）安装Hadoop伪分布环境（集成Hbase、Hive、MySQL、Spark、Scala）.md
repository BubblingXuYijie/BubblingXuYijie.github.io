---
title: Deepin（Linux）安装Hadoop伪分布环境（集成Hbase、Hive、MySQL、Spark、Scala）
date: 2021-04-08 12:14:25
categories: Linux
tags:
    - Deepin
    - Ubuntu
    - Debian
    - Hadoop
    - Hbase
    - Hive
    - Spark
cover: https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/DebianHadoop0.png
---
# 安装版本
Hadoop3.2.2、Hbase2.4.17、Hive3.1.2、MySQL8.0.24、Spark3.1.1、Scala2.13.5
（即使可以，也不推荐使用apt安装，因为版本会报错，所有我下面均使用压缩包来安装）

# 下载所有环境
[Hadoop3.2.2下载](https://archive.apache.org/dist/hadoop/common/hadoop-3.2.2/) bfsu这个镜像下载最快![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/HadoopDownload.png)
[Hbase2.4.17下载](https://dlcdn.apache.org/hbase/2.4.17/) 选择bin.tar.gz

[Hive3.1.2下载](https://mirrors.bfsu.edu.cn/apache/hive/hive-3.1.2/)选择bin.tar.gz
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/DebianHadoop1.png)
[Spark下载地址](https://spark.apache.org/downloads.html)记得第2栏选择Hadoop版本，本教程是3.2
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/DebianHadoop2.png)
[Scala2.13.5下载地址](https://www.scala-lang.org/download/) 进去官网后拉到最底部，选择第一个
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/DebianHadoop3.png)


[MySQL下载地址](https://dev.mysql.com/downloads/mysql/)

这里一定要选择这个Linux Generic
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/DebianHadoop4.png)
下载前面这两个，64位和32位根据自己电脑情况来选


# 开启Deepin或Ubuntu（我用的是虚拟机）
我用的是Deepin20（Debian10 Buster库）好看吧， 还有“QQ2008”
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/DebianHadoop5.png)
#  安装Hadoop
#### 1、安装和配置ssh
首先在终端输入sudo apt-get update来更新一下apt的包列表（apt代表赋予管理员权限，建议每句命令都加上）
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/DebianHadoop6.png)
输入sudo apt-get install openssh-server回车，再输入sudo apt-get install openssh-client回车（Deepin可能已经预装了ssh，不过再输入一下确认一下也没什么）
Ubuntu安装是如下界面
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/DebianHadoop7.png)
Deepin已经安装过是如下界面![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/DebianHadoop8.png)
配置ssh无密码自动登录（很重要）

```powershell
$ ssh localhost              #登陆SSH，第一次登陆输入yes
```

```powershell
$ exit                       #退出登录的ssh localhost
```

```powershell
$ cd ~/.ssh/                 #如果没法进入该目录，执行一次ssh localhost
```

```powershell
$ ssh-keygen -t rsa
```

输入完$ ssh-keygen -t rsa 语句后，需要连续敲击三次回车
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/DebianHadoop9.png)

```powershell
$ cat ./id_rsa.pub >> ./authorized_keys #加入授权
```

```powershell
$ ssh localhost     #此时已不需密码即可登录localhost，并可见下图的Welcome
```
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/DebianHadoop10.png)

#### 2、安装和配置Java（一定要安装Java11版本之前的，不然Spark和Scala会报错，我们安装Java8）

输入apt-cache search openjdk ,这是查看可以安装的Java版本有哪些，查询结果显示如下，我们可以看到，又openjdk-8-jdk
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/DebianHadoop11.png)![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/DebianHadoop47.png)
接着，我们输入sudo apt-get install openjdk-8-jdk 回车，等待下载就安装成功了

```powershell
$ sudo apt-get install openjdk-8-jdk
```

无论是Deepin还是Ubuntu，apt安装的Java都存在系统盘的usr/lib.jvm里
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/DebianHadoop12.png)
最后配置Java的环境变量

我们在终端中输入vim ~/.bashrc ,会进入一个文档，也许会出现这个页面，按一下键盘上的E就能进入文档了

```powershell
$ vim ~/.bashrc
```

![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/DebianHadoop13.png)
进入文档后是这样的
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/DebianHadoop14.png)
我这边配置了本教程所有的环境变量，大家之间粘贴进去，后面就不用再配置了

注意！！！vim的操作和普通文本编辑器不同！！！

vim进去以后，先按一下键盘上的A，进入编辑模式，然后再右键，点粘贴，或者是CTRL+SHIFT+V粘贴，粘贴上去以后，按一下键盘上的“ESC”键，然后按“SHIFT+ZZ”就保存并退出到终端了
```bash
export JAVA_HOME=/usr/lib/jvm/jdk8
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib:${HADOOP_HOME}/bin:$HIVE_HOME/bin:$CALSSPATH
export SCALA_HOME=/usr/local/scala
export SPARK_HOME=/usr/local/spark
export HADOOP_HOME=/usr/local/hadoop
export HIVE_HOME=/usr/local/hive
export HBASE_HOME=/usr/local/hbase
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$HBASE_HOME/bin:${JAVA_HOME}/bin:$JRE_HOME/bin:$HIVE_HOME/bin:$SCALA_HOME/bin:$SPARK_HOME/bin:$SPARK_HOME/sbin
```

然后输入source ~/.bashrc，这个意思是使配置的环境变量立即生效
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/DebianHadoop15.png)
检查Java版本，安装并配置成功
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/DebianHadoop16.png)
#### 3、安装Hadoop
我们把下载好的Hadoop安装包放在“下载”文件夹里，Deepin下载的默认路径就是这个

在这个目录下右键选择“在终端中打开”
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/DebianHadoop17.png)
在打开的终端中输入sudo tar -zxvf  hadoop-3.2.2.tar -C /usr/local

将下载的Hadoop压缩包解压到usr/local/hadoop-3.2.2文件夹下，hadoop-3.2.2.tar.gz是下载的文件名

```powershell
sudo tar -zxvf  hadoop-3.2.2.tar.gz -C /usr/local
```
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/DebianHadoop18.png)
然后将hadoop-3.2.2重命名为hadoop，依次执行下列命令

```powershell
$ cd /usr/local
```

```powershell
$ sudo mv ./hadoop-3.2.2 ./hadoop
```

查看文件夹，已经命名为hadoop
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/DebianHadoop19.png)
你们的hadoop文件夹上肯定会带有一个黄色的锁🔒，还需要赋予hadoop文件夹权限，避免以后出现问题，接着执行下列命令，🔒就消失了，以后安装Hbase、Hive等，也是执行下列命令赋予权限，把文件夹名修改一下就行

```powershell
$ sudo chmod 777 -R hadoop
```

伪分布需要配置三个文件

1、/hadoop/etc/hadoop路径下的hadoop-env.sh，右键打开方式选文本编辑器，添加下面代码进去
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/DebianHadoop20.png)


```bash
export HADOOP_OS_TYPE=${HADOOP_OS_TYPE:-$(uname -s)}
export JAVA_HOME=/usr/lib/jvm/jdk8
export HADOOP_CONF_DIR=/usr/local/hadoop/etc/hadoop
export HADOOP_SSH_OPTS="-p 22"
export HADOOP_CLASSPATH=.:$CLASSPATH:$HADOOP_CLASSPATH:$HADOOP_HOME/bin
```

![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/DebianHadoop21.png)
2、/hadoop/etc/hadoop路径下的core-site.xml，添加下面代码进去

```xml
<configuration>
<property>
    <name>fs.default.name</name>
    <value>hdfs://localhost:9000</value>
    <description>HDFS的URI，文件系统://namenode标识:端口号</description>
</property>

<property>
    <name>hadoop.tmp.dir</name>
    <value>file:/usr/local/hadoop/tmp</value>
    <description>namenode上本地的hadoop临时文件夹</description>
</property>

<property>
	<name>dfs.permissions.enabled</name>
	<value>false</value>
</property>

<property>
	<name>hadoop.proxyuser.root.hosts</name>
	<value>*</value>
</property>

<property>
	<name>hadoop.proxyuser.root.groups</name>
	<value>*</value>
</property>

</configuration>
```
3、/hadoop/etc/hadoop路径下的hdfs-site.xml，添加下面代码进去

```xml
<configuration>
      <property>
	<name>dfs.namenode.name.dir</name>
    <value>file:/usr/local/hadoop/tmp/dfs/name</value>
    <description>namenode上存储hdfs名字空间元数据 </description> 
</property>

<property>
    <name>dfs.datanode.data.dir</name>
    <value>file:/usr/local/hadoop/tmp/dfs/data</value>
    <description>datanode上数据块的物理存储位置</description>
</property>

<property>
    <name>dfs.checkpoint.dir</name>
    <value>file:/usr/local/hadoop/tmp/dfs/snn</value>
    <description>secondary namenode 的位置</description>
</property>

<property>
    <name>dfs.checkpoint.edits.dir</name>
    <value>file:/usr/local/hadoop/tmp/dfs/snn</value>
    <description>secondary namenode 的位置</description>
</property>

<property>
    <name>dfs.replication</name>
    <value>1</value>
    <description>副本个数，配置默认是3,应小于datanode机器数量</description>
</property>
</configuration>
```
Hadoop 的运行方式是由配置文件决定的，因此如果需要从伪分布式模式切换回非分布式模式，需要删除 core-site.xml 中的配置项。此外，伪分布式虽然只需要配置 fs.defaultFS 和 dfs.replication 就可以运行（参考官方教程），不过若没有配置 hadoop.tmp.dir 参数，则默认使用的临时目录为 /tmp/hadoo-hadoop，而这个目录在重启时有可能被系统清理掉，导致必须重新执行 format 才行。所以我们进行了设置，同时也指定 dfs.namenode.name.dir 和 dfs.datanode.data.dir，否则在接下来的步骤中可能会出错。

配置完之后，执行根节点的格式化，在hadoop文件夹下右键，“在终端中打开”
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/DebianHadoop22.png)

输入bin/hdfs namenode -format
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/DebianHadoop23.png)
格式化根节点只在第一次配置完之后执行一次，切勿多次手动格式化，如果多次手动格式化导致DataNode或NameNode无法启动，请关闭Hadoop服务，删除hadoop文件夹下的这三个文件夹后，重新格式化根节点
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/DebianHadoop24.png)
####  4、启动Hadoop
我们已经配置好环境变量，所以可以在终端中直接输入start-dfs.sh来启动Hadoop的服务
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/DebianHadoop25.png)
注意！！！！今后启动不再使用start-dfs.sh命令，因为这个命令没有启动所有的Hadoop服务，以后做项目或者配置其他环境会报错，记住，以后启动Hadoop使用start-all.sh这个命令

打开浏览器，输入localhost:9870（Hadoop3的默认端口）
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/DebianHadoop26.png)

关闭Hadoop服务的命令是stop-dfs.sh

#  安装Hbase
#### 1、解压Hbase
参考解压hadoop进行解压
重命名文件夹
赋予权限

本教程安装的所有的环境都解压在/usr/local里，后面不在赘述解压过程
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/DebianHadoop27.png)
#### 2、配置Hbase伪分布环境
将下面代码复制到Hbase文件夹里的conf文件夹里的hbase-site.xml文件中

```xml
<configuration>
  <property>
    <name>hbase.rootdir</name>
<!-- 端口要和Hadoop的fs.defaultFS端口一致-->
    <value>hdfs://localhost:9000/hbase</value>
  </property>
  <property>
<!-- 是否分布式部署 -->
    <name>hbase.cluster.distributed</name>
    <value>true</value>
  </property>
  <property>
    <name>hbase.zookeeper.quorum</name>
    <value>localhost</value>
  </property>
  <property>
    <name>dfs.replication</name>
    <value>1</value>
  </property>
  <property>
    <name>hbase.unsafe.stream.capability.enforce</name>
    <value>false</value>
  </property>
  <property>
    <name>hbase.zookeeper.property.clientPort</name>
     <value>2182</value>                                                                                                                                             
  </property> 
  <property>
    <name>hbase.master.info.port</name>
    <value>60010</value>                                                                                                                                             
  </property>
</configuration>
```
将下面代码复制到Hbase文件夹里的conf文件夹里的hbase-env.sh文件中

```xml
export JAVA_HOME=/usr/lib/jvm/jdk8
export HBASE_MANAGES_ZK=true
export HBASE_CLASSPATH=/usr/local/hbase/conf
export HBASE_DISABLE_HADOOP_CLASSPATH_LOOKUP="true"
```
然后就可以启动Hbase了，注意，启动Hbase之前先启动Hadoop，关闭Hbase的命令是stop-hbase.sh
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/DebianHadoop27.png)
可以localhost:60010访问Hbase页面
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/DebianHadoop28.png)
####  3、hbase shell的基本操作
输入hbase shell可以打开hbase shell
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/DebianHadoop29.png)
建个表
查看表属性
添加一个学生信息
查看学生信息
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/DebianHadoop30.png)
退出shell输入exit回车
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/DebianHadoop31.png)

#  安装Hive
####  1、安装Hive
解压过程不赘述，请看安装Haoop板块介绍的方法

修改Hive/conf文件夹下的hive-site.xml，如果没有就在该文件夹下右键，“在终端中打开”，使用下面命令新建一个hive-site.xml

```xml
$ sudo gedit hive-site.xml
```
或者直接右键，新建文本文档，后缀修改为xml
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/DebianHadoop32.png)
在hive-site.xml中这样写，此配置为MySQL8.0.24的配置，如果版本低于8，则不是这样配置

```xml
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
    <property>
	  <name>javax.jdo.option.ConnectionURL</name>
	  <value>jdbc:mysql://localhost:3306/metastore?createDatabaseIfNotExist=true</value>
	  <description>JDBC connect string for a JDBC metastore</description>
	</property>

	<property>
	  <name>javax.jdo.option.ConnectionDriverName</name>
	  <value>com.mysql.cj.jdbc.Driver</value>
	  <description>Driver class name for a JDBC metastore</description>
	</property>

 <property>
    <name>javax.jdo.option.ConnectionUserName</name>
    <value>hive</value>
    <description>username to use against metastore database</description>
  </property>
  <property>
    <name>javax.jdo.option.ConnectionPassword</name>
    <value>hive</value>
    <description>password to use against metastore database</description>
  </property>

<property>
<name>hive.metastore.warehouse.dir</name>
<value>/usr/local/hive/warehouse</value>
</property>

<property>
<name>hive.exec.scratchdir</name>
<value>/usr/local/hive/tmp</value>
</property>

<!-- 日志目录 -->
<property>
<name>hive.querylog.location</name>
<value>/usr/local/hive/log</value>
</property>

<!-- 设置metastore的节点信息 -->
<property>
<name>hive.metastore.uris</name>
<value>thrift://linux100:9083</value>
</property>

<!-- 客户端远程连接的端口 -->
<property> 
<name>hive.server2.thrift.port</name> 
<value>10000</value>
</property>
<property> 
<name>hive.server2.thrift.bind.host</name> 
<value>0.0.0.0</value>
</property>
<property>
<name>hive.server2.webui.host</name>
<value>0.0.0.0</value>
</property>

<!-- hive服务的页面的端口 -->
<property>
<name>hive.server2.webui.port</name>
<value>10002</value>
</property>

<property> 
<name>hive.server2.long.polling.timeout</name> 
<value>5000</value>                               
</property>

<property>
<name>hive.server2.enable.doAs</name>
<value>true</value>
</property>

<property>
<name>datanucleus.autoCreateSchema</name>
<value>false</value>
</property>

<property>
<name>datanucleus.fixedDatastore</name>
<value>true</value>
</property>

<property>
<name>hive.execution.engine</name>
<value>mr</value>
</property>

<!-- 添加如下配置信息，就可以实现显示当前数据库，以及查询表的头信息配置 -->
	 <property>
      <name>hive.cli.print.header</name>
      <value>true</value>
    </property>

    <property>
      <name>hive.cli.print.current.db</name>
      <value>true</value>
    </property>

</configuration>
```

修改hive/conf文件夹下的hive-env.sh，添加下面语句

```bash
export HADOOP_HOME=/usr/local/hadoop/
export HIVE_CONF_DIR=/usr/local/hive/conf/
export HIVE_AUX_JARS_PATH=/usr/local/hive/lib
```
#### 2、安装MySQL
MySQL可以使用apt安装，Deepin默认的MySQL就是最新版，Ubuntu我不知道是不是，可以使用下面命令看一下版本
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/DebianHadoop33.png)
然后使用下面命令安装MySQL

```powershell
sudo apt-get install mysql-server
```
安装完成后查看MySQL版本
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/DebianHadoop34.png)
接下来还要安装一个mysql-connector-java
[下载地址](https://dev.mysql.com/downloads/connector/j/)
Deepin和UOS选Debian，Ubuntu选Ubuntu
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/DebianHadoop35.png)
下载完成后，文件是deb格式，直接双击点击安装就行了，我的因为安装过了，只有更新可以点
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/DebianHadoop36.png)
在终端输入以下命令启动MySQL服务

```powershell
sudo systemctl start mysql
```
执行sudo mysql_secure_installation

```powershell
sudo mysql_secure_installation
```
为MySQL设置密码（一定要设置，而且密码不能为空，不然后面报错）
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/DebianHadoop37.png)
然后下面会让你输入很多Y/N，全部输入Y回车

完成所有设置后，以root用户登录MySQL。 在终端中，键入以下命令：mysql -u root -p，输入 root用户的密码，然后按Enter ，登陆进MySQL
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/DebianHadoop38.png)
新建hive数据库，别忘了加分号
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/DebianHadoop39.png)
然后依次输入下面语句，意思是将所有数据库的所有表的所有权限赋给hive用户，后面的hive是配置hive-site.xml中配置的连接密码

```sql
CREATE USER 'hive'@'localhost' IDENTIFIED BY 'hive';
GRANT ALL ON *.* TO 'hive'@'localhost';
```
然后输入下面语句

```sql
flush privileges;  #刷新mysql系统权限关系表
```
#### 3、启动Hive
启动Hive之前，先运行start-all.sh启动Hadoop集群，然后输入hive启动

如果输入hive以后出现下面报错
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/DebianHadoop40.png)
则输入hdfs dfsadmin -safemode leave关闭安全模式，再运行hive即可
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/DebianHadoop41.png)
启动成功，输入exitl;可退出hive shell
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/DebianHadoop42.png)
#  安装Spark
####  1、安装Scala语言支持
解压不再赘述，请看安装Haoop板块介绍的方法，安装好查看版本
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/DebianHadoop43.png)
####  2、安装Spark
配置spark/conf文件夹下的spark-env.sh

在最后一行下面添加

```bash
export JAVA_HOME=/usr/lib/jvm/jdk8
export HADOOP_HOME=/usr/local/hadoop
export HADOOP_CONF_DIR=/usr/local/hadoop/etc/hadoop
export SCALA_HOME=/usr/local/scala
export SPARK_HOME=/usr/local/spark
export SPARK_MASTER_IP=127.0.0.1
export SPARK_MASTER_PORT=7077
export SPARK_MASTER_WEBUI_PORT=8099
export SPARK_WORKER_CORES=3
export SPARK_WORKER_INSTANCES=1
export SPARK_WORKER_MEMORY=5G
export SPARK_WORKER_WEBUI_PORT=8081
export SPARK_EXECUTOR_CORES=1
export SPARK_EXECUTOR_MEMORY=1G
export LD_LIBRARY_PATH=${LD_LIBRARY_PATH}:$HADOOP_HOME/lib/native
```
配置spark/conf文件夹下的worker，就在最后一行下面加上一个localhost即可（Spark基于hadoop3.2版本的配置文件从slaves变成了worker，旧版本为slaves）
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/DebianHadoop44.png)
####  3、启动Spark
启动Spark之前，同样要先保证Hadoop集群已经启动

启动Spark使用start-master.sh和start-slaves.sh

然后输入jps，查看，多出了Master和Worker
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/DebianHadoop45.png)
#### 4、开启Spark Shell
spark shell的退出命令是 :quit
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/DebianHadoop46.png)
# 结语
4.10：好了这篇文章到这就暂时结束了，Linux真sd，这系非人类，虽然早就配好了，但是等我心情好了再写吧
4.29：心情不错，正在更新
4.29：一口气写完

后续会写一个WordCount实例的教程

WordCount实例教程已写完，项目打包成jar包，可以在hadoop下运行

传送门：[MapReduce基础编程，实现WordCount实例](https://blog.csdn.net/qq_48922459/article/details/116274365?spm=1001.2014.3001.5501)

