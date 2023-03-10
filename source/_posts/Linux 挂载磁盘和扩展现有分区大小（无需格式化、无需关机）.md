---
title: Linux 挂载磁盘和扩展现有分区大小（无需格式化、无需关机）
date: 2021-12-30 10:25:28
categories: Linux
tags:
    - Linux
cover: https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/LinuxLogo.jpg
---
# 前言

<font color=#999AAA >Centos、Ubuntu、Debian，xfs 和 ext 文件系统都可以，我全都在虚拟机试过一遍，先讲挂载磁盘，因为扩容也需要挂载的前几步操作。</font>

<hr style=" border:solid; width:100px; height:1px;" color=#000000 size=1">



# 一、磁盘挂载


<font color=#999AAA >如果是虚拟机，先在虚拟机设置里面增加磁盘，点右边扩展，然后输入的最大磁盘大小，原本20G，输入40G，也就是扩容20G，完成后开启虚拟机，在系统里继续配置。
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/挂载磁盘0.png)



<font color=#999AAA >进入系统以后，输入`df -lh`查看现有分区挂载状态，再输入`fdisk -l`查看所拥有的的磁盘，我只有一个磁盘，就是 /dev/sda，现在是42.9G，可以看到磁盘容量已经增加了（之前是22G）

![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/挂载磁盘1.png)

<font color=#999AAA >如果我们要挂载这块磁盘到一个新的文件夹叫 /new，我们这样操作

<font color=#999AAA >输入`fdisk /dev/sda`回车，（/dev/sda换成你的磁盘名）

<font color=#999AAA >会提示你一些信息，输入 “p”查看分区表信息，输入“n”创建分区，慢点回车，一直到看见 `分区 * 已设置为 Linux 类型，大小设为 * GiB` （有些系统这个提示是英文的），就输入 wq ，回车

==提示：如果不想把这块磁盘的空间全部分区，就在“Last 扇区”那里，不要直接回车，输入数字修改大小再回车==

![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/挂载磁盘2.png)

<font color=#999AAA > 再次输入 `fdisk -l`你会发现下面比之前多了一个 sda4（你们的可能不叫sda4哈，反正就是多了一个），就是刚刚新建的分区
![在这里插入图片描述](https://img-blog.csdnimg.cn/a22a92a8cb5140feb2414fb88bec5fd3.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA57mB5Y2O5bC95aS05ruh5piv5q6H,size_11,color_FFFFFF,t_70,g_se,x_16)
<font color=#999AAA > 如果要把这个新分区挂载到文件夹的话就输入 `mount /dev/sdb4 /new`，后面的 /new 是你要挂载的文件夹，再运行`vi /etc/fstab`，设置开机自动挂载，然后`vi /etc/fstab`在文件里添加以下内容`/dev/sdb    /data    ext4    defaults    0 0`，完成

<font color=#999AAA > 如果是扩容就不需要上面那一步操作了，直接进行下一步

<hr style=" border:solid; width:100px; height:1px;" color=#000000 size=1">

# 二、分区扩容


<font color=#999AAA >运行 `vgdisplay`，显示卷组信息，查看VG Name，下面要用，这时 Free PE 为0

![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/挂载磁盘3.png)


<font color=#999AAA >运行`partprobe - inform the os of partition table changes` 来使分区表生效（提示没有那个文件或目录不用管）

<font color=#999AAA >然后运行`pvcreate /dev/sda4` 将刚才的分区初始化为物理卷，以便被 LVM 使用


![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/挂载磁盘4.png)


<font color=#999AAA >运行`vgextend centos /dev/sda4`，  扩展卷组（上面的VG Name换成你自己的，我的叫centos，你们自己换一下）

<font color=#999AAA >再次运行`vgdisplay`查看卷组信息，发现Free PE有空间了

![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/挂载磁盘5.png)
<hr style=" border:solid; width:100px; height:1px;" color=#000000 size=1">


<font color=#999AAA >然后开始扩容啦，有两种写法，百分比和直接指定扩容大小（下面有图）




<font color=#999AAA >指定扩展大小：
`lvextend -L+9.9G /dev/mapper/centos-root /dev/sda4`
意思是为 /dev/mapper/centos-root 增加10G空间，这个 /dev/mapper/centos-root 是 df -lh 那一步你要扩容的分区名称，你们自己改一下，==直接写10G系统会判定超出磁盘大小，所以写9.9G，建议下面那种写100%不浪费空间==

<hr style=" border:solid; width:100px; height:1px;" color=#000000 size=1">


<font color=#999AAA >按照百分比扩展：
`lvextend -l +100%FREE /dev/mapper/centos-root /dev/sda4`
意思是把 /dev/sda4 的全部空间扩容给 /dev/mapper/centos-root ，注意命令大小写

<hr style=" border:solid; width:100px; height:1px;" color=#000000 size=1">



<font color=#999AAA >最后一步，看一下你的分区文件系统，运行`cat /etc/fstab`，我的是 xfs



如果是ext文件系统：`resize2fs /dev/mapper/centos-root`
如果是XFS文件系统：`xfs_growfs /dev/mapper/centos-root`

<font color=#999AAA >然后`df -lh`发现扩容成功

![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/挂载磁盘6.png)







<hr style=" border:solid; width:100px; height:1px;" color=#000000 size=1">

# 总结
