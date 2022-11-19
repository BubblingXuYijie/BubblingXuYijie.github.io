---
title: AWS Airport 搭建日志
date: 2022-04-10 12:37:34
categories: Linux
tags:
    - Linux
    - AWS
    - 虚拟专用网络
    - 网络
cover: https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/shadowsocksLogo.png
---
# 前言
`仅供学习交流，为了我们大家好，不要在网络上传播本篇内容`

==注意，下面的服务器操作请务必使用root用户，或者每句命令前加 sudo==


---

# 一、安装谷歌BBR加速

> TCP BBR是谷歌出品的TCP拥塞控制算法。BBR目的是要尽量跑满带宽，并且尽量不要有排队的情况。BBR可以起到单边加速TCP连接的效果。Google提交到Linux主线并发表在ACM queue期刊上的TCP-BBR拥塞控制算法。继承了Google“先在生产环境上部署，再开源和发论文”的研究传统。
>
>TCP-BBR已经再YouTube服务器和Google跨数据中心的内部广域网(B4)上部署。由此可见出该算法的前途。TCP-BBR的目标就是最大化利用网络上瓶颈链路的带宽。一条网络链路就像一条水管，要想最大化利用这条水管，最好的办法就是给这跟水管灌满水。
>
>BBR解决了两个问题：在有一定丢包率的网络链路上充分利用带宽。非常适合高延迟，高带宽的网络链路。降低网络链路上的buffer占用率，从而降低延迟。非常适合慢速接入网络的用户。Google 在 2016年9月份开源了他们的优化网络拥堵算法BBR，最新版本的 Linux内核(4.9-rc8)中已经集成了该算法。对于TCP单边加速，并非所有人都很熟悉，不过有另外一个大名鼎鼎的商业软件“锐速”，相信很多人都清楚。特别是对于使用国外服务器或者VPS的人来说，效果更佳。


https://raw.githubusercontent.com/teddysun/across/master/bbr.sh

有些服务器系统在进行`wget --no-check-certificate https://github.com/teddysun/across/raw/master/bbr.sh && chmod +x bbr.sh && ./bbr.sh`这行命令时会出现权限不足的问题，所以可以使用上面的地址自行新建一个 `bbr.sh` 文件


```bash
[ec2-user@ip-172-31-29-157 ~]$ vim bbr.sh # 新建一个bbr.sh，文件内容在上方链接
[ec2-user@ip-172-31-29-157 ~]$ chmod 777 bbr.sh # 赋予最高权限
[ec2-user@ip-172-31-29-157 ~]$ sh bbr.sh # 运行sh文件，如果不使用root权限会报错
[Error] This script must be run as root
[ec2-user@ip-172-31-29-157 ~]$ su root
Password: 
su: Authentication failure
[ec2-user@ip-172-31-29-157 ~]$ sudo -s # 切换到root用户，两种方式，上面那个需要密码
[root@ip-172-31-29-157 ec2-user] ls
bbr.sh
[root@ip-172-31-29-157 ec2-user] sh bbr.sh # 运行sh文件
---------- System Information ----------
 OS      : Amazon Linux 2
 Arch    : x86_64 (64 Bit)
 Kernel  : 5.10.102-99.473.amzn2.x86_64
----------------------------------------
 Automatically enable TCP BBR script

 URL: https://teddysun.com/489.html
----------------------------------------

Press any key to start...or Press Ctrl+C to cancel

[Info] The kernel version is greater than 4.9, directly setting TCP BBR...
[Info] Setting TCP BBR completed...
[root@ip-172-31-29-157 ec2-user] lsmod | grep bbr #检查谁否安装成功
tcp_bbr                20480  1

```




# 二、安装小飞机

```bash
[root@ip-172-31-29-157 ec2-user] wget --no-check-certificate -O shadowsocks-all.sh https://raw.githubusercontent.com/teddysun/shadowsocks_install/master/shadowsocks-all.sh #下载小飞机运行脚本
--2022-04-10 03:12:12--  https://raw.githubusercontent.com/teddysun/shadowsocks_install/master/shadowsocks-all.sh
Resolving raw.githubusercontent.com (raw.githubusercontent.com)... 185.199.108.133, 185.199.109.133, 185.199.110.133, ...
Connecting to raw.githubusercontent.com (raw.githubusercontent.com)|185.199.108.133|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 46729 (46K) [text/plain]
Saving to: ‘shadowsocks-all.sh’

100%[=========================================================================================================================================================================>] 46,729      --.-K/s   in 0.001s  

2022-04-10 03:12:12 (67.2 MB/s) - ‘shadowsocks-all.sh’ saved [46729/46729]

[root@ip-172-31-29-157 ec2-user] chmod 777 shadowsocks-all.sh # 赋予脚本最高权限
[root@ip-172-31-29-157 ec2-user] ./shadowsocks-all.sh 2>＆1 | tee shadowsocks-all.log # 执行脚
Which Shadowsocks server you select: #选择客户端语言，我们一般选python
1) Shadowsocks-Python
2) ShadowsocksR
3) Shadowsocks-Go
4) Shadowsocks-libev
1

You choose = Shadowsocks-Python

Please enter password for Shadowsocks-Python #设置连接密码
XuYijie312416

password = XuYijie312416

Please enter a port for Shadowsocks-Python [1-65535] # 设置连接端口（不要忘记防火墙放行）
5000

port = 5000

Please select stream cipher for Shadowsocks-Python: # 选择加密方式
1) aes-256-gcm
2) aes-192-gcm
3) aes-128-gcm
4) aes-256-ctr
5) aes-192-ctr
6) aes-128-ctr
7) aes-256-cfb
8) aes-192-cfb
9) aes-128-cfb
10) camellia-128-cfb
11) camellia-192-cfb
12) camellia-256-cfb
13) xchacha20-ietf-poly1305
14) chacha20-ietf-poly1305
15) chacha20-ietf
16) chacha20
17) salsa20
18) rc4-md5
1

cipher = aes-256-gcm


Press any key to start...or Press Ctrl+C to cancel # 按下回车，可能出现以下报错
[Info] Checking the EPEL repository...
[Error] Install EPEL repository failed, please check it.


#解决方案，修复epel


[root@ip-172-31-29-157 ec2-user] yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
Loaded plugins: extras_suggestions, langpacks, priorities, update-motd
epel-release-latest-7.noarch.rpm                                                                                                                                                            |  15 kB  00:00:00     
Examining /var/tmp/yum-root-oCtaN1/epel-release-latest-7.noarch.rpm: epel-release-7-14.noarch
Marking /var/tmp/yum-root-oCtaN1/epel-release-latest-7.noarch.rpm to be installed
Resolving Dependencies
--> Running transaction check
---> Package epel-release.noarch 0:7-14 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

===================================================================================================================================================================================================================
 Package                                           Arch                                        Version                                    Repository                                                          Size
===================================================================================================================================================================================================================
Installing:
 epel-release                                      noarch                                      7-14                                       /epel-release-latest-7.noarch                                       25 k

Transaction Summary
===================================================================================================================================================================================================================
Install  1 Package

Total size: 25 k
Installed size: 25 k
Is this ok [y/d/N]: y
Downloading packages:
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : epel-release-7-14.noarch                                                                                                                                                                        1/1 
  Verifying  : epel-release-7-14.noarch                                                                                                                                                                        1/1 

Installed:
  epel-release.noarch 0:7-14                                                                                                                                                                                       

Complete!
[root@ip-172-31-29-157 ec2-user]# yum-config-manager --enable epel
Loaded plugins: extras_suggestions, langpacks, priorities, update-motd
=================================================================================================== repo: epel ====================================================================================================
[epel]
async = True
bandwidth = 0
base_persistdir = /var/lib/yum/repos/x86_64/2
baseurl = 
cache = 0
cachedir = /var/cache/yum/x86_64/2/epel
check_config_file_age = True
compare_providers_priority = 80
cost = 1000
deltarpm_metadata_percentage = 100
deltarpm_percentage = 
enabled = True
enablegroups = True
exclude = 
failovermethod = priority
ftp_disable_epsv = False
gpgcadir = /var/lib/yum/repos/x86_64/2/epel/gpgcadir
gpgcakey = 
gpgcheck = True
gpgdir = /var/lib/yum/repos/x86_64/2/epel/gpgdir
gpgkey = file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7
hdrdir = /var/cache/yum/x86_64/2/epel/headers
http_caching = all
includepkgs = 
ip_resolve = 
keepalive = True
keepcache = False
mddownloadpolicy = sqlite
mdpolicy = group:small
mediaid = 
metadata_expire = 21600
metadata_expire_filter = read-only:present
metalink = https://mirrors.fedoraproject.org/metalink?repo=epel-7&arch=x86_64&infra=$infra&content=$contentdir
minrate = 0
mirrorlist = 
mirrorlist_expire = 86400
name = Extra Packages for Enterprise Linux 7 - x86_64
old_base_cache_dir = 
password = 
persistdir = /var/lib/yum/repos/x86_64/2/epel
pkgdir = /var/cache/yum/x86_64/2/epel/packages
priority = 99
proxy = False
proxy_dict = 
proxy_password = 
proxy_username = 
repo_gpgcheck = False
report_instanceid = False
retries = 7
skip_if_unavailable = False
ssl_check_cert_permissions = True
sslcacert = 
sslclientcert = 
sslclientkey = 
sslverify = True
throttle = 0
timeout = 5.0
ui_id = epel/x86_64
ui_repoid_vars = releasever,
   basearch
username = 

# 然后再运行一次小飞机脚本，成功

[root@ip-172-31-29-157 ec2-user] ./shadowsocks-all.sh 2>＆1 | tee shadowsocks-all.log
Which Shadowsocks server you select:
1) Shadowsocks-Python
2) ShadowsocksR
3) Shadowsocks-Go
4) Shadowsocks-libev
1

You choose = Shadowsocks-Python

Please enter password for Shadowsocks-Python
XuYijie312416

password = XuYijie312416

Please enter a port for Shadowsocks-Python [1-65535]
5000

port = 5000

Please select stream cipher for Shadowsocks-Python:
1) aes-256-gcm
2) aes-192-gcm
3) aes-128-gcm
4) aes-256-ctr
5) aes-192-ctr
6) aes-128-ctr
7) aes-256-cfb
8) aes-192-cfb
9) aes-128-cfb
10) camellia-128-cfb
11) camellia-192-cfb
12) camellia-256-cfb
13) xchacha20-ietf-poly1305
14) chacha20-ietf-poly1305
15) chacha20-ietf
16) chacha20
17) salsa20
18) rc4-md5
1

cipher = aes-256-gcm


Press any key to start...or Press Ctrl+C to cancel
[Info] Checking the EPEL repository...
[Info] Checking the EPEL repository complete...
[Info] Starting to install package unzip
[Info] Starting to install package gzip
[Info] Starting to install package openssl
[Info] Starting to install package openssl-devel
[Info] Starting to install package gcc
[Info] Starting to install package python
[Info] Starting to install package python-devel
[Info] Starting to install package python-setuptools
[Info] Starting to install package pcre
[Info] Starting to install package pcre-devel
[Info] Starting to install package libtool
[Info] Starting to install package libevent
[Info] Starting to install package autoconf
[Info] Starting to install package automake
[Info] Starting to install package make
[Info] Starting to install package curl
[Info] Starting to install package curl-devel
[Info] Starting to install package zlib-devel
[Info] Starting to install package perl
[Info] Starting to install package perl-devel
[Info] Starting to install package cpio
[Info] Starting to install package expat-devel
[Info] Starting to install package gettext-devel
[Info] Starting to install package libev-devel
[Info] Starting to install package c-ares-devel
[Info] Starting to install package git
[Info] Starting to install package qrencode
shadowsocks-master.zip not found, download now...
shadowsocks-python not found, download now...
libsodium-1.0.18.tar.gz not found, download now...
checking build system type... x86_64-pc-linux-gnu
checking host system type... x86_64-pc-linux-gnu
checking for a BSD-compatible install... /bin/install -c
checking whether build environment is sane... yes
checking for a thread-safe mkdir -p... /bin/mkdir -p
checking for gawk... gawk
checking whether make sets $(MAKE)... yes
checking whether make supports nested variables... yes
checking whether UID '0' is supported by ustar format... yes
checking whether GID '0' is supported by ustar format... yes
checking how to create a ustar tar archive... gnutar
checking whether make supports nested variables... (cached) yes
checking whether to enable maintainer-specific portions of Makefiles... no
checking whether make supports the include directive... yes (GNU style)
checking for gcc... gcc
checking whether the C compiler works... yes
checking for C compiler default output file name... a.out
checking for suffix of executables... 
checking whether we are cross compiling... no
checking for suffix of object files... o
checking whether we are using the GNU C compiler... yes
checking whether gcc accepts -g... yes
checking for gcc option to accept ISO C89... none needed
checking whether gcc understands -c and -o together... yes
checking dependency style of gcc... gcc3
checking for a sed that does not truncate output... /bin/sed
checking how to run the C preprocessor... gcc -E
checking for grep that handles long lines and -e... /bin/grep
checking for egrep... /bin/grep -E
checking whether gcc is Clang... no
checking whether pthreads work with -pthread... yes
checking for joinable pthread attribute... PTHREAD_CREATE_JOINABLE
checking whether more special flags are required for pthreads... no
checking for PTHREAD_PRIO_INHERIT... yes
checking for gcc option to accept ISO C99... none needed
checking dependency style of gcc... gcc3
checking for ANSI C header files... yes
checking for sys/types.h... yes
checking for sys/stat.h... yes
checking for stdlib.h... yes
checking for string.h... yes
checking for memory.h... yes
checking for strings.h... yes
checking for inttypes.h... yes
checking for stdint.h... yes
checking for unistd.h... yes
checking minix/config.h usability... no
checking minix/config.h presence... no
checking for minix/config.h... no
checking whether it is safe to define __EXTENSIONS__... yes
checking for variable-length arrays... yes
checking for __wasi__ defined... no
checking for _FORTIFY_SOURCE defined... no
checking whether C compiler accepts -D_FORTIFY_SOURCE=2... yes
checking whether C compiler accepts -fvisibility=hidden... yes
checking whether C compiler accepts -fPIC... yes
checking whether C compiler accepts -fPIE... yes
checking whether the linker accepts -pie... yes
checking whether C compiler accepts -fno-strict-aliasing... yes
checking whether C compiler accepts -fno-strict-overflow... yes
checking whether C compiler accepts -fstack-protector... yes
checking whether the linker accepts -fstack-protector... yes
checking whether C compiler accepts -g -O2 -pthread -fvisibility=hidden -fPIC -fPIE -fno-strict-aliasing -fno-strict-overflow -fstack-protector -Wall... yes
checking whether C compiler accepts -g -O2 -pthread -fvisibility=hidden -fPIC -fPIE -fno-strict-aliasing -fno-strict-overflow -fstack-protector -Wextra... yes
checking for clang... no
checking whether C compiler accepts -g -O2 -pthread -fvisibility=hidden -fPIC -fPIE -fno-strict-aliasing -fno-strict-overflow -fstack-protector -Wextra -Wbad-function-cast... yes
checking whether C compiler accepts -g -O2 -pthread -fvisibility=hidden -fPIC -fPIE -fno-strict-aliasing -fno-strict-overflow -fstack-protector -Wextra -Wbad-function-cast -Wcast-qual... yes
checking whether C compiler accepts -g -O2 -pthread -fvisibility=hidden -fPIC -fPIE -fno-strict-aliasing -fno-strict-overflow -fstack-protector -Wextra -Wbad-function-cast -Wcast-qual -Wdiv-by-zero... yes
checking whether C compiler accepts -g -O2 -pthread -fvisibility=hidden -fPIC -fPIE -fno-strict-aliasing -fno-strict-overflow -fstack-protector -Wextra -Wbad-function-cast -Wcast-qual -Wdiv-by-zero -Wduplicated-branches... yes
checking whether C compiler accepts -g -O2 -pthread -fvisibility=hidden -fPIC -fPIE -fno-strict-aliasing -fno-strict-overflow -fstack-protector -Wextra -Wbad-function-cast -Wcast-qual -Wdiv-by-zero -Wduplicated-branches -Wduplicated-cond... yes
checking whether C compiler accepts -g -O2 -pthread -fvisibility=hidden -fPIC -fPIE -fno-strict-aliasing -fno-strict-overflow -fstack-protector -Wextra -Wbad-function-cast -Wcast-qual -Wdiv-by-zero -Wduplicated-branches -Wduplicated-cond -Wfloat-equal... yes
checking whether C compiler accepts -g -O2 -pthread -fvisibility=hidden -fPIC -fPIE -fno-strict-aliasing -fno-strict-overflow -fstack-protector -Wextra -Wbad-function-cast -Wcast-qual -Wdiv-by-zero -Wduplicated-branches -Wduplicated-cond -Wfloat-equal -Wformat=2... yes
checking whether C compiler accepts -g -O2 -pthread -fvisibility=hidden -fPIC -fPIE -fno-strict-aliasing -fno-strict-overflow -fstack-protector -Wextra -Wbad-function-cast -Wcast-qual -Wdiv-by-zero -Wduplicated-branches -Wduplicated-cond -Wfloat-equal -Wformat=2 -Wlogical-op... yes
checking whether C compiler accepts -g -O2 -pthread -fvisibility=hidden -fPIC -fPIE -fno-strict-aliasing -fno-strict-overflow -fstack-protector -Wextra -Wbad-function-cast -Wcast-qual -Wdiv-by-zero -Wduplicated-branches -Wduplicated-cond -Wfloat-equal -Wformat=2 -Wlogical-op -Wmaybe-uninitialized... yes
checking whether C compiler accepts -g -O2 -pthread -fvisibility=hidden -fPIC -fPIE -fno-strict-aliasing -fno-strict-overflow -fstack-protector -Wextra -Wbad-function-cast -Wcast-qual -Wdiv-by-zero -Wduplicated-branches -Wduplicated-cond -Wfloat-equal -Wformat=2 -Wlogical-op -Wmaybe-uninitialized -Wmisleading-indentation... yes
checking whether C compiler accepts -g -O2 -pthread -fvisibility=hidden -fPIC -fPIE -fno-strict-aliasing -fno-strict-overflow -fstack-protector -Wextra -Wbad-function-cast -Wcast-qual -Wdiv-by-zero -Wduplicated-branches -Wduplicated-cond -Wfloat-equal -Wformat=2 -Wlogical-op -Wmaybe-uninitialized -Wmisleading-indentation -Wmissing-declarations... yes
checking whether C compiler accepts -g -O2 -pthread -fvisibility=hidden -fPIC -fPIE -fno-strict-aliasing -fno-strict-overflow -fstack-protector -Wextra -Wbad-function-cast -Wcast-qual -Wdiv-by-zero -Wduplicated-branches -Wduplicated-cond -Wfloat-equal -Wformat=2 -Wlogical-op -Wmaybe-uninitialized -Wmisleading-indentation -Wmissing-declarations -Wmissing-prototypes... yes
checking whether C compiler accepts -g -O2 -pthread -fvisibility=hidden -fPIC -fPIE -fno-strict-aliasing -fno-strict-overflow -fstack-protector -Wextra -Wbad-function-cast -Wcast-qual -Wdiv-by-zero -Wduplicated-branches -Wduplicated-cond -Wfloat-equal -Wformat=2 -Wlogical-op -Wmaybe-uninitialized -Wmisleading-indentation -Wmissing-declarations -Wmissing-prototypes -Wnested-externs... yes
checking whether C compiler accepts -g -O2 -pthread -fvisibility=hidden -fPIC -fPIE -fno-strict-aliasing -fno-strict-overflow -fstack-protector -Wextra -Wbad-function-cast -Wcast-qual -Wdiv-by-zero -Wduplicated-branches -Wduplicated-cond -Wfloat-equal -Wformat=2 -Wlogical-op -Wmaybe-uninitialized -Wmisleading-indentation -Wmissing-declarations -Wmissing-prototypes -Wnested-externs -Wno-type-limits... yes
checking whether C compiler accepts -g -O2 -pthread -fvisibility=hidden -fPIC -fPIE -fno-strict-aliasing -fno-strict-overflow -fstack-protector -Wextra -Wbad-function-cast -Wcast-qual -Wdiv-by-zero -Wduplicated-branches -Wduplicated-cond -Wfloat-equal -Wformat=2 -Wlogical-op -Wmaybe-uninitialized -Wmisleading-indentation -Wmissing-declarations -Wmissing-prototypes -Wnested-externs -Wno-type-limits -Wno-unknown-pragmas... yes
checking whether C compiler accepts -g -O2 -pthread -fvisibility=hidden -fPIC -fPIE -fno-strict-aliasing -fno-strict-overflow -fstack-protector -Wextra -Wbad-function-cast -Wcast-qual -Wdiv-by-zero -Wduplicated-branches -Wduplicated-cond -Wfloat-equal -Wformat=2 -Wlogical-op -Wmaybe-uninitialized -Wmisleading-indentation -Wmissing-declarations -Wmissing-prototypes -Wnested-externs -Wno-type-limits -Wno-unknown-pragmas -Wnormalized=id... yes
checking whether C compiler accepts -g -O2 -pthread -fvisibility=hidden -fPIC -fPIE -fno-strict-aliasing -fno-strict-overflow -fstack-protector -Wextra -Wbad-function-cast -Wcast-qual -Wdiv-by-zero -Wduplicated-branches -Wduplicated-cond -Wfloat-equal -Wformat=2 -Wlogical-op -Wmaybe-uninitialized -Wmisleading-indentation -Wmissing-declarations -Wmissing-prototypes -Wnested-externs -Wno-type-limits -Wno-unknown-pragmas -Wnormalized=id -Wnull-dereference... yes
checking whether C compiler accepts -g -O2 -pthread -fvisibility=hidden -fPIC -fPIE -fno-strict-aliasing -fno-strict-overflow -fstack-protector -Wextra -Wbad-function-cast -Wcast-qual -Wdiv-by-zero -Wduplicated-branches -Wduplicated-cond -Wfloat-equal -Wformat=2 -Wlogical-op -Wmaybe-uninitialized -Wmisleading-indentation -Wmissing-declarations -Wmissing-prototypes -Wnested-externs -Wno-type-limits -Wno-unknown-pragmas -Wnormalized=id -Wnull-dereference -Wold-style-declaration... yes
checking whether C compiler accepts -g -O2 -pthread -fvisibility=hidden -fPIC -fPIE -fno-strict-aliasing -fno-strict-overflow -fstack-protector -Wextra -Wbad-function-cast -Wcast-qual -Wdiv-by-zero -Wduplicated-branches -Wduplicated-cond -Wfloat-equal -Wformat=2 -Wlogical-op -Wmaybe-uninitialized -Wmisleading-indentation -Wmissing-declarations -Wmissing-prototypes -Wnested-externs -Wno-type-limits -Wno-unknown-pragmas -Wnormalized=id -Wnull-dereference -Wold-style-declaration -Wpointer-arith... yes
checking whether C compiler accepts -g -O2 -pthread -fvisibility=hidden -fPIC -fPIE -fno-strict-aliasing -fno-strict-overflow -fstack-protector -Wextra -Wbad-function-cast -Wcast-qual -Wdiv-by-zero -Wduplicated-branches -Wduplicated-cond -Wfloat-equal -Wformat=2 -Wlogical-op -Wmaybe-uninitialized -Wmisleading-indentation -Wmissing-declarations -Wmissing-prototypes -Wnested-externs -Wno-type-limits -Wno-unknown-pragmas -Wnormalized=id -Wnull-dereference -Wold-style-declaration -Wpointer-arith -Wredundant-decls... yes
checking whether C compiler accepts -g -O2 -pthread -fvisibility=hidden -fPIC -fPIE -fno-strict-aliasing -fno-strict-overflow -fstack-protector -Wextra -Wbad-function-cast -Wcast-qual -Wdiv-by-zero -Wduplicated-branches -Wduplicated-cond -Wfloat-equal -Wformat=2 -Wlogical-op -Wmaybe-uninitialized -Wmisleading-indentation -Wmissing-declarations -Wmissing-prototypes -Wnested-externs -Wno-type-limits -Wno-unknown-pragmas -Wnormalized=id -Wnull-dereference -Wold-style-declaration -Wpointer-arith -Wredundant-decls -Wrestrict... yes
checking whether C compiler accepts -g -O2 -pthread -fvisibility=hidden -fPIC -fPIE -fno-strict-aliasing -fno-strict-overflow -fstack-protector -Wextra -Wbad-function-cast -Wcast-qual -Wdiv-by-zero -Wduplicated-branches -Wduplicated-cond -Wfloat-equal -Wformat=2 -Wlogical-op -Wmaybe-uninitialized -Wmisleading-indentation -Wmissing-declarations -Wmissing-prototypes -Wnested-externs -Wno-type-limits -Wno-unknown-pragmas -Wnormalized=id -Wnull-dereference -Wold-style-declaration -Wpointer-arith -Wredundant-decls -Wrestrict -Wshorten-64-to-32... no
checking whether C compiler accepts -g -O2 -pthread -fvisibility=hidden -fPIC -fPIE -fno-strict-aliasing -fno-strict-overflow -fstack-protector -Wextra -Wbad-function-cast -Wcast-qual -Wdiv-by-zero -Wduplicated-branches -Wduplicated-cond -Wfloat-equal -Wformat=2 -Wlogical-op -Wmaybe-uninitialized -Wmisleading-indentation -Wmissing-declarations -Wmissing-prototypes -Wnested-externs -Wno-type-limits -Wno-unknown-pragmas -Wnormalized=id -Wnull-dereference -Wold-style-declaration -Wpointer-arith -Wredundant-decls -Wrestrict -Wsometimes-uninitialized... no
checking whether C compiler accepts -g -O2 -pthread -fvisibility=hidden -fPIC -fPIE -fno-strict-aliasing -fno-strict-overflow -fstack-protector -Wextra -Wbad-function-cast -Wcast-qual -Wdiv-by-zero -Wduplicated-branches -Wduplicated-cond -Wfloat-equal -Wformat=2 -Wlogical-op -Wmaybe-uninitialized -Wmisleading-indentation -Wmissing-declarations -Wmissing-prototypes -Wnested-externs -Wno-type-limits -Wno-unknown-pragmas -Wnormalized=id -Wnull-dereference -Wold-style-declaration -Wpointer-arith -Wredundant-decls -Wrestrict -Wstrict-prototypes... yes
checking whether C compiler accepts -g -O2 -pthread -fvisibility=hidden -fPIC -fPIE -fno-strict-aliasing -fno-strict-overflow -fstack-protector -Wextra -Wbad-function-cast -Wcast-qual -Wdiv-by-zero -Wduplicated-branches -Wduplicated-cond -Wfloat-equal -Wformat=2 -Wlogical-op -Wmaybe-uninitialized -Wmisleading-indentation -Wmissing-declarations -Wmissing-prototypes -Wnested-externs -Wno-type-limits -Wno-unknown-pragmas -Wnormalized=id -Wnull-dereference -Wold-style-declaration -Wpointer-arith -Wredundant-decls -Wrestrict -Wstrict-prototypes -Wswitch-enum... yes
checking whether C compiler accepts -g -O2 -pthread -fvisibility=hidden -fPIC -fPIE -fno-strict-aliasing -fno-strict-overflow -fstack-protector -Wextra -Wbad-function-cast -Wcast-qual -Wdiv-by-zero -Wduplicated-branches -Wduplicated-cond -Wfloat-equal -Wformat=2 -Wlogical-op -Wmaybe-uninitialized -Wmisleading-indentation -Wmissing-declarations -Wmissing-prototypes -Wnested-externs -Wno-type-limits -Wno-unknown-pragmas -Wnormalized=id -Wnull-dereference -Wold-style-declaration -Wpointer-arith -Wredundant-decls -Wrestrict -Wstrict-prototypes -Wswitch-enum -Wvariable-decl... no
checking whether C compiler accepts -g -O2 -pthread -fvisibility=hidden -fPIC -fPIE -fno-strict-aliasing -fno-strict-overflow -fstack-protector -Wextra -Wbad-function-cast -Wcast-qual -Wdiv-by-zero -Wduplicated-branches -Wduplicated-cond -Wfloat-equal -Wformat=2 -Wlogical-op -Wmaybe-uninitialized -Wmisleading-indentation -Wmissing-declarations -Wmissing-prototypes -Wnested-externs -Wno-type-limits -Wno-unknown-pragmas -Wnormalized=id -Wnull-dereference -Wold-style-declaration -Wpointer-arith -Wredundant-decls -Wrestrict -Wstrict-prototypes -Wswitch-enum -Wwrite-strings... yes
checking whether the linker accepts -Wl,-z,relro... yes
checking whether the linker accepts -Wl,-z,now... yes
checking whether the linker accepts -Wl,-z,noexecstack... yes
checking whether segmentation violations can be caught when using the C compiler... yes
checking whether SIGABRT can be caught when using the C compiler... yes
checking for thread local storage (TLS) class... _Thread_local
thread local storage is supported
checking whether C compiler accepts -ftls-model=local-dynamic... yes
checking how to print strings... printf
checking for a sed that does not truncate output... (cached) /bin/sed
checking for fgrep... /bin/grep -F
checking for ld used by gcc... /bin/ld
checking if the linker (/bin/ld) is GNU ld... yes
checking for BSD- or MS-compatible name lister (nm)... /bin/nm -B
checking the name lister (/bin/nm -B) interface... BSD nm
checking whether ln -s works... yes
checking the maximum length of command line arguments... 1572864
checking how to convert x86_64-pc-linux-gnu file names to x86_64-pc-linux-gnu format... func_convert_file_noop
checking how to convert x86_64-pc-linux-gnu file names to toolchain format... func_convert_file_noop
checking for /bin/ld option to reload object files... -r
checking for objdump... objdump
checking how to recognize dependent libraries... pass_all
checking for dlltool... no
checking how to associate runtime and link libraries... printf %s\n
checking for ar... ar
checking for archiver @FILE support... @
checking for strip... strip
checking for ranlib... ranlib
checking command to parse /bin/nm -B output from gcc object... ok
checking for sysroot... no
checking for a working dd... /bin/dd
checking how to truncate binary pipes... /bin/dd bs=4096 count=1
checking for mt... no
checking if : is a manifest tool... no
checking for dlfcn.h... yes
checking for objdir... .libs
checking if gcc supports -fno-rtti -fno-exceptions... no
checking for gcc option to produce PIC... -fPIC -DPIC
checking if gcc PIC flag -fPIC -DPIC works... yes
checking if gcc static flag -static works... no
checking if gcc supports -c -o file.o... yes
checking if gcc supports -c -o file.o... (cached) yes
checking whether the gcc linker (/bin/ld -m elf_x86_64) supports shared libraries... yes
checking whether -lc should be explicitly linked in... no
checking dynamic linker characteristics... GNU/Linux ld.so
checking how to hardcode library paths into programs... immediate
checking whether stripping libraries is possible... yes
checking if libtool supports shared libraries... yes
checking whether to build shared libraries... yes
checking whether to build static libraries... yes
checking for ar... (cached) ar
checking whether C compiler accepts -mmmx... yes
checking for MMX instructions set... yes
checking whether C compiler accepts -mmmx... (cached) yes
checking whether C compiler accepts -msse2... yes
checking for SSE2 instructions set... yes
checking whether C compiler accepts -msse2... (cached) yes
checking whether C compiler accepts -msse3... yes
checking for SSE3 instructions set... yes
checking whether C compiler accepts -msse3... (cached) yes
checking whether C compiler accepts -mssse3... yes
checking for SSSE3 instructions set... yes
checking whether C compiler accepts -mssse3... (cached) yes
checking whether C compiler accepts -msse4.1... yes
checking for SSE4.1 instructions set... yes
checking whether C compiler accepts -msse4.1... (cached) yes
checking whether C compiler accepts -mavx... yes
checking for AVX instructions set... yes
checking whether C compiler accepts -mavx... (cached) yes
checking whether C compiler accepts -mavx2... yes
checking for AVX2 instructions set... yes
checking whether C compiler accepts -mavx2... (cached) yes
checking if _mm256_broadcastsi128_si256 is correctly defined... yes
checking whether C compiler accepts -mavx512f... yes
checking for AVX512F instructions set... yes
checking whether C compiler accepts -mavx512f... (cached) yes
checking whether C compiler accepts -maes... yes
checking whether C compiler accepts -mpclmul... yes
checking for AESNI instructions set and PCLMULQDQ... yes
checking whether C compiler accepts -maes... (cached) yes
checking whether C compiler accepts -mpclmul... (cached) yes
checking whether C compiler accepts -mrdrnd... yes
checking for RDRAND... yes
checking whether C compiler accepts -mrdrnd... (cached) yes
checking sys/mman.h usability... yes
checking sys/mman.h presence... yes
checking for sys/mman.h... yes
checking sys/random.h usability... yes
checking sys/random.h presence... yes
checking for sys/random.h... yes
checking intrin.h usability... no
checking intrin.h presence... no
checking for intrin.h... no
checking if _xgetbv() is available... no
checking for inline... inline
checking whether byte ordering is bigendian... (cached) no
checking whether __STDC_LIMIT_MACROS is required... no
checking whether we can use inline asm code... yes
no
checking whether we can use x86_64 asm code... yes
checking whether we can assemble AVX opcodes... yes
checking for 128-bit arithmetic... yes
checking for cpuid instruction... yes
checking if the .private_extern asm directive is supported... no
checking if the .hidden asm directive is supported... yes
checking if weak symbols are supported... yes
checking if data alignment is required... no
checking if atomic operations are supported... yes
checking for size_t... yes
checking for working alloca.h... yes
checking for alloca... yes
checking for arc4random... no
checking for arc4random_buf... no
checking for mmap... yes
checking for mlock... yes
checking for madvise... yes
checking for mprotect... yes
checking for getrandom with a standard API... yes
checking for getrandom... yes
checking for getentropy with a standard API... yes
checking for getentropy... yes
checking for posix_memalign... yes
checking for getpid... yes
checking for nanosleep... yes
checking for memset_s... no
checking for explicit_bzero... yes
checking for explicit_memset... no
checking if gcc/ld supports -Wl,--output-def... no
checking that generated files are newer than configure... done
configure: creating ./config.status
config.status: creating Makefile
config.status: creating builds/Makefile
config.status: creating contrib/Makefile
config.status: creating dist-build/Makefile
config.status: creating libsodium.pc
config.status: creating libsodium-uninstalled.pc
config.status: creating msvc-scripts/Makefile
config.status: creating src/Makefile
config.status: creating src/libsodium/Makefile
config.status: creating src/libsodium/include/Makefile
config.status: creating src/libsodium/include/sodium/version.h
config.status: creating test/default/Makefile
config.status: creating test/Makefile
config.status: executing depfiles commands
config.status: executing libtool commands
Making all in builds
make[1]: Entering directory `/home/ec2-user/libsodium-1.0.18/builds'
make[1]: Nothing to be done for `all'.
make[1]: Leaving directory `/home/ec2-user/libsodium-1.0.18/builds'
Making all in contrib
make[1]: Entering directory `/home/ec2-user/libsodium-1.0.18/contrib'
make[1]: Nothing to be done for `all'.
make[1]: Leaving directory `/home/ec2-user/libsodium-1.0.18/contrib'
Making all in dist-build
make[1]: Entering directory `/home/ec2-user/libsodium-1.0.18/dist-build'
make[1]: Nothing to be done for `all'.
make[1]: Leaving directory `/home/ec2-user/libsodium-1.0.18/dist-build'
Making all in msvc-scripts
make[1]: Entering directory `/home/ec2-user/libsodium-1.0.18/msvc-scripts'
make[1]: Nothing to be done for `all'.
make[1]: Leaving directory `/home/ec2-user/libsodium-1.0.18/msvc-scripts'
Making all in src
make[1]: Entering directory `/home/ec2-user/libsodium-1.0.18/src'
Making all in libsodium
make[2]: Entering directory `/home/ec2-user/libsodium-1.0.18/src/libsodium'
Making all in include
make[3]: Entering directory `/home/ec2-user/libsodium-1.0.18/src/libsodium/include'
make[3]: Nothing to be done for `all'.
make[3]: Leaving directory `/home/ec2-user/libsodium-1.0.18/src/libsodium/include'
make[3]: Entering directory `/home/ec2-user/libsodium-1.0.18/src/libsodium'
  CC       crypto_aead/chacha20poly1305/sodium/libsodium_la-aead_chacha20poly1305.lo
  CC       crypto_aead/xchacha20poly1305/sodium/libsodium_la-aead_xchacha20poly1305.lo
  CC       crypto_auth/libsodium_la-crypto_auth.lo
  CC       crypto_auth/hmacsha256/libsodium_la-auth_hmacsha256.lo
  CC       crypto_auth/hmacsha512/libsodium_la-auth_hmacsha512.lo
  CC       crypto_auth/hmacsha512256/libsodium_la-auth_hmacsha512256.lo
  CC       crypto_box/libsodium_la-crypto_box.lo
  CC       crypto_box/libsodium_la-crypto_box_easy.lo
  CC       crypto_box/libsodium_la-crypto_box_seal.lo
  CC       crypto_box/curve25519xsalsa20poly1305/libsodium_la-box_curve25519xsalsa20poly1305.lo
  CC       crypto_core/ed25519/ref10/libsodium_la-ed25519_ref10.lo
  CC       crypto_core/hchacha20/libsodium_la-core_hchacha20.lo
  CC       crypto_core/hsalsa20/ref2/libsodium_la-core_hsalsa20_ref2.lo
  CC       crypto_core/hsalsa20/libsodium_la-core_hsalsa20.lo
  CC       crypto_core/salsa/ref/libsodium_la-core_salsa_ref.lo
  CC       crypto_generichash/libsodium_la-crypto_generichash.lo
  CC       crypto_generichash/blake2b/libsodium_la-generichash_blake2.lo
  CC       crypto_generichash/blake2b/ref/libsodium_la-blake2b-compress-ref.lo
  CC       crypto_generichash/blake2b/ref/libsodium_la-blake2b-ref.lo
  CC       crypto_generichash/blake2b/ref/libsodium_la-generichash_blake2b.lo
  CC       crypto_hash/libsodium_la-crypto_hash.lo
  CC       crypto_hash/sha256/libsodium_la-hash_sha256.lo
  CC       crypto_hash/sha256/cp/libsodium_la-hash_sha256_cp.lo
  CC       crypto_hash/sha512/libsodium_la-hash_sha512.lo
  CC       crypto_hash/sha512/cp/libsodium_la-hash_sha512_cp.lo
  CC       crypto_kdf/blake2b/libsodium_la-kdf_blake2b.lo
  CC       crypto_kdf/libsodium_la-crypto_kdf.lo
  CC       crypto_kx/libsodium_la-crypto_kx.lo
  CC       crypto_onetimeauth/libsodium_la-crypto_onetimeauth.lo
  CC       crypto_onetimeauth/poly1305/libsodium_la-onetimeauth_poly1305.lo
  CC       crypto_onetimeauth/poly1305/donna/libsodium_la-poly1305_donna.lo
  CC       crypto_pwhash/argon2/libsodium_la-argon2-core.lo
  CC       crypto_pwhash/argon2/libsodium_la-argon2-encoding.lo
  CC       crypto_pwhash/argon2/libsodium_la-argon2-fill-block-ref.lo
  CC       crypto_pwhash/argon2/libsodium_la-argon2.lo
  CC       crypto_pwhash/argon2/libsodium_la-blake2b-long.lo
  CC       crypto_pwhash/argon2/libsodium_la-pwhash_argon2i.lo
  CC       crypto_pwhash/argon2/libsodium_la-pwhash_argon2id.lo
  CC       crypto_pwhash/libsodium_la-crypto_pwhash.lo
  CC       crypto_scalarmult/libsodium_la-crypto_scalarmult.lo
  CC       crypto_scalarmult/curve25519/ref10/libsodium_la-x25519_ref10.lo
  CC       crypto_scalarmult/curve25519/libsodium_la-scalarmult_curve25519.lo
  CC       crypto_secretbox/libsodium_la-crypto_secretbox.lo
  CC       crypto_secretbox/libsodium_la-crypto_secretbox_easy.lo
  CC       crypto_secretbox/xsalsa20poly1305/libsodium_la-secretbox_xsalsa20poly1305.lo
  CC       crypto_secretstream/xchacha20poly1305/libsodium_la-secretstream_xchacha20poly1305.lo
  CC       crypto_shorthash/libsodium_la-crypto_shorthash.lo
  CC       crypto_shorthash/siphash24/libsodium_la-shorthash_siphash24.lo
  CC       crypto_shorthash/siphash24/ref/libsodium_la-shorthash_siphash24_ref.lo
  CC       crypto_sign/libsodium_la-crypto_sign.lo
  CC       crypto_sign/ed25519/libsodium_la-sign_ed25519.lo
  CC       crypto_sign/ed25519/ref10/libsodium_la-keypair.lo
  CC       crypto_sign/ed25519/ref10/libsodium_la-open.lo
  CC       crypto_sign/ed25519/ref10/libsodium_la-sign.lo
  CC       crypto_stream/chacha20/libsodium_la-stream_chacha20.lo
  CC       crypto_stream/chacha20/ref/libsodium_la-chacha20_ref.lo
  CC       crypto_stream/libsodium_la-crypto_stream.lo
  CC       crypto_stream/salsa20/libsodium_la-stream_salsa20.lo
  CC       crypto_stream/xsalsa20/libsodium_la-stream_xsalsa20.lo
  CC       crypto_verify/sodium/libsodium_la-verify.lo
  CC       randombytes/libsodium_la-randombytes.lo
  CC       sodium/libsodium_la-codecs.lo
  CC       sodium/libsodium_la-core.lo
  CC       sodium/libsodium_la-runtime.lo
  CC       sodium/libsodium_la-utils.lo
  CC       sodium/libsodium_la-version.lo
  CPPAS    crypto_stream/salsa20/xmm6/libsodium_la-salsa20_xmm6-asm.lo
  CC       crypto_stream/salsa20/xmm6/libsodium_la-salsa20_xmm6.lo
  CC       crypto_scalarmult/curve25519/sandy2x/libsodium_la-curve25519_sandy2x.lo
  CC       crypto_scalarmult/curve25519/sandy2x/libsodium_la-fe51_invert.lo
  CC       crypto_scalarmult/curve25519/sandy2x/libsodium_la-fe_frombytes_sandy2x.lo
  CPPAS    crypto_scalarmult/curve25519/sandy2x/libsodium_la-sandy2x.lo
  CC       crypto_box/curve25519xchacha20poly1305/libsodium_la-box_curve25519xchacha20poly1305.lo
  CC       crypto_box/curve25519xchacha20poly1305/libsodium_la-box_seal_curve25519xchacha20poly1305.lo
  CC       crypto_core/ed25519/libsodium_la-core_ed25519.lo
  CC       crypto_core/ed25519/libsodium_la-core_ristretto255.lo
  CC       crypto_pwhash/scryptsalsa208sha256/libsodium_la-crypto_scrypt-common.lo
  CC       crypto_pwhash/scryptsalsa208sha256/libsodium_la-scrypt_platform.lo
  CC       crypto_pwhash/scryptsalsa208sha256/libsodium_la-pbkdf2-sha256.lo
  CC       crypto_pwhash/scryptsalsa208sha256/libsodium_la-pwhash_scryptsalsa208sha256.lo
  CC       crypto_pwhash/scryptsalsa208sha256/nosse/libsodium_la-pwhash_scryptsalsa208sha256_nosse.lo
  CC       crypto_scalarmult/ed25519/ref10/libsodium_la-scalarmult_ed25519_ref10.lo
  CC       crypto_scalarmult/ristretto255/ref10/libsodium_la-scalarmult_ristretto255_ref10.lo
  CC       crypto_secretbox/xchacha20poly1305/libsodium_la-secretbox_xchacha20poly1305.lo
  CC       crypto_shorthash/siphash24/libsodium_la-shorthash_siphashx24.lo
  CC       crypto_shorthash/siphash24/ref/libsodium_la-shorthash_siphashx24_ref.lo
  CC       crypto_sign/ed25519/ref10/libsodium_la-obsolete.lo
  CC       crypto_stream/salsa2012/ref/libsodium_la-stream_salsa2012_ref.lo
  CC       crypto_stream/salsa2012/libsodium_la-stream_salsa2012.lo
  CC       crypto_stream/salsa208/ref/libsodium_la-stream_salsa208_ref.lo
  CC       crypto_stream/salsa208/libsodium_la-stream_salsa208.lo
  CC       crypto_stream/xchacha20/libsodium_la-stream_xchacha20.lo
  CC       randombytes/sysrandom/libsodium_la-randombytes_sysrandom.lo
  CC       crypto_aead/aes256gcm/aesni/libaesni_la-aead_aes256gcm_aesni.lo
  CCLD     libaesni.la
  CC       crypto_onetimeauth/poly1305/sse2/libsse2_la-poly1305_sse2.lo
  CC       crypto_pwhash/scryptsalsa208sha256/sse/libsse2_la-pwhash_scryptsalsa208sha256_sse.lo
  CCLD     libsse2.la
  CC       crypto_generichash/blake2b/ref/libssse3_la-blake2b-compress-ssse3.lo
  CC       crypto_pwhash/argon2/libssse3_la-argon2-fill-block-ssse3.lo
  CC       crypto_stream/chacha20/dolbeau/libssse3_la-chacha20_dolbeau-ssse3.lo
  CCLD     libssse3.la
  CC       crypto_generichash/blake2b/ref/libsse41_la-blake2b-compress-sse41.lo
  CCLD     libsse41.la
  CC       crypto_generichash/blake2b/ref/libavx2_la-blake2b-compress-avx2.lo
  CC       crypto_pwhash/argon2/libavx2_la-argon2-fill-block-avx2.lo
  CC       crypto_stream/chacha20/dolbeau/libavx2_la-chacha20_dolbeau-avx2.lo
  CC       crypto_stream/salsa20/xmm6int/libavx2_la-salsa20_xmm6int-avx2.lo
  CCLD     libavx2.la
  CC       crypto_pwhash/argon2/libavx512f_la-argon2-fill-block-avx512f.lo
  CCLD     libavx512f.la
  CC       randombytes/internal/librdrand_la-randombytes_internal_random.lo
  CCLD     librdrand.la
  CCLD     libsodium.la
make[3]: Leaving directory `/home/ec2-user/libsodium-1.0.18/src/libsodium'
make[2]: Leaving directory `/home/ec2-user/libsodium-1.0.18/src/libsodium'
make[2]: Entering directory `/home/ec2-user/libsodium-1.0.18/src'
make[2]: Nothing to be done for `all-am'.
make[2]: Leaving directory `/home/ec2-user/libsodium-1.0.18/src'
make[1]: Leaving directory `/home/ec2-user/libsodium-1.0.18/src'
Making all in test
make[1]: Entering directory `/home/ec2-user/libsodium-1.0.18/test'
Making all in default
make[2]: Entering directory `/home/ec2-user/libsodium-1.0.18/test/default'
make[2]: Nothing to be done for `all'.
make[2]: Leaving directory `/home/ec2-user/libsodium-1.0.18/test/default'
make[2]: Entering directory `/home/ec2-user/libsodium-1.0.18/test'
make[2]: Nothing to be done for `all-am'.
make[2]: Leaving directory `/home/ec2-user/libsodium-1.0.18/test'
make[1]: Leaving directory `/home/ec2-user/libsodium-1.0.18/test'
make[1]: Entering directory `/home/ec2-user/libsodium-1.0.18'
make[1]: Nothing to be done for `all-am'.
make[1]: Leaving directory `/home/ec2-user/libsodium-1.0.18'
Making install in builds
make[1]: Entering directory `/home/ec2-user/libsodium-1.0.18/builds'
make[2]: Entering directory `/home/ec2-user/libsodium-1.0.18/builds'
make[2]: Nothing to be done for `install-exec-am'.
make[2]: Nothing to be done for `install-data-am'.
make[2]: Leaving directory `/home/ec2-user/libsodium-1.0.18/builds'
make[1]: Leaving directory `/home/ec2-user/libsodium-1.0.18/builds'
Making install in contrib
make[1]: Entering directory `/home/ec2-user/libsodium-1.0.18/contrib'
make[2]: Entering directory `/home/ec2-user/libsodium-1.0.18/contrib'
make[2]: Nothing to be done for `install-exec-am'.
make[2]: Nothing to be done for `install-data-am'.
make[2]: Leaving directory `/home/ec2-user/libsodium-1.0.18/contrib'
make[1]: Leaving directory `/home/ec2-user/libsodium-1.0.18/contrib'
Making install in dist-build
make[1]: Entering directory `/home/ec2-user/libsodium-1.0.18/dist-build'
make[2]: Entering directory `/home/ec2-user/libsodium-1.0.18/dist-build'
make[2]: Nothing to be done for `install-exec-am'.
make[2]: Nothing to be done for `install-data-am'.
make[2]: Leaving directory `/home/ec2-user/libsodium-1.0.18/dist-build'
make[1]: Leaving directory `/home/ec2-user/libsodium-1.0.18/dist-build'
Making install in msvc-scripts
make[1]: Entering directory `/home/ec2-user/libsodium-1.0.18/msvc-scripts'
make[2]: Entering directory `/home/ec2-user/libsodium-1.0.18/msvc-scripts'
make[2]: Nothing to be done for `install-exec-am'.
make[2]: Nothing to be done for `install-data-am'.
make[2]: Leaving directory `/home/ec2-user/libsodium-1.0.18/msvc-scripts'
make[1]: Leaving directory `/home/ec2-user/libsodium-1.0.18/msvc-scripts'
Making install in src
make[1]: Entering directory `/home/ec2-user/libsodium-1.0.18/src'
Making install in libsodium
make[2]: Entering directory `/home/ec2-user/libsodium-1.0.18/src/libsodium'
Making install in include
make[3]: Entering directory `/home/ec2-user/libsodium-1.0.18/src/libsodium/include'
make[4]: Entering directory `/home/ec2-user/libsodium-1.0.18/src/libsodium/include'
make[4]: Nothing to be done for `install-exec-am'.
 /bin/mkdir -p '/usr/include'
 /bin/mkdir -p '/usr/include/sodium'
 /bin/install -c -m 644  sodium/core.h sodium/crypto_aead_aes256gcm.h sodium/crypto_aead_chacha20poly1305.h sodium/crypto_aead_xchacha20poly1305.h sodium/crypto_auth.h sodium/crypto_auth_hmacsha256.h sodium/crypto_auth_hmacsha512.h sodium/crypto_auth_hmacsha512256.h sodium/crypto_box.h sodium/crypto_box_curve25519xchacha20poly1305.h sodium/crypto_box_curve25519xsalsa20poly1305.h sodium/crypto_core_ed25519.h sodium/crypto_core_ristretto255.h sodium/crypto_core_hchacha20.h sodium/crypto_core_hsalsa20.h sodium/crypto_core_salsa20.h sodium/crypto_core_salsa2012.h sodium/crypto_core_salsa208.h sodium/crypto_generichash.h sodium/crypto_generichash_blake2b.h sodium/crypto_hash.h sodium/crypto_hash_sha256.h sodium/crypto_hash_sha512.h sodium/crypto_kdf.h sodium/crypto_kdf_blake2b.h sodium/crypto_kx.h sodium/crypto_onetimeauth.h sodium/crypto_onetimeauth_poly1305.h sodium/crypto_pwhash.h sodium/crypto_pwhash_argon2i.h sodium/crypto_pwhash_argon2id.h sodium/crypto_pwhash_scryptsalsa208sha256.h sodium/crypto_scalarmult.h sodium/crypto_scalarmult_curve25519.h sodium/crypto_scalarmult_ed25519.h sodium/crypto_scalarmult_ristretto255.h sodium/crypto_secretbox.h sodium/crypto_secretbox_xchacha20poly1305.h sodium/crypto_secretbox_xsalsa20poly1305.h sodium/crypto_secretstream_xchacha20poly1305.h '/usr/include/sodium'
 /bin/mkdir -p '/usr/include/sodium'
 /bin/install -c -m 644  sodium/crypto_shorthash.h sodium/crypto_shorthash_siphash24.h sodium/crypto_sign.h sodium/crypto_sign_ed25519.h sodium/crypto_sign_edwards25519sha512batch.h sodium/crypto_stream.h sodium/crypto_stream_chacha20.h sodium/crypto_stream_salsa20.h sodium/crypto_stream_salsa2012.h sodium/crypto_stream_salsa208.h sodium/crypto_stream_xchacha20.h sodium/crypto_stream_xsalsa20.h sodium/crypto_verify_16.h sodium/crypto_verify_32.h sodium/crypto_verify_64.h sodium/export.h sodium/randombytes.h sodium/randombytes_internal_random.h sodium/randombytes_sysrandom.h sodium/runtime.h sodium/utils.h '/usr/include/sodium'
 /bin/install -c -m 644  sodium.h '/usr/include/.'
 /bin/mkdir -p '/usr/include'
 /bin/mkdir -p '/usr/include/sodium'
 /bin/install -c -m 644  sodium/version.h '/usr/include/sodium'
make[4]: Leaving directory `/home/ec2-user/libsodium-1.0.18/src/libsodium/include'
make[3]: Leaving directory `/home/ec2-user/libsodium-1.0.18/src/libsodium/include'
make[3]: Entering directory `/home/ec2-user/libsodium-1.0.18/src/libsodium'
make[4]: Entering directory `/home/ec2-user/libsodium-1.0.18/src/libsodium'
 /bin/mkdir -p '/usr/lib'
 /bin/sh ../../libtool   --mode=install /bin/install -c   libsodium.la '/usr/lib'
libtool: install: /bin/install -c .libs/libsodium.so.23.3.0 /usr/lib/libsodium.so.23.3.0
libtool: install: (cd /usr/lib && { ln -s -f libsodium.so.23.3.0 libsodium.so.23 || { rm -f libsodium.so.23 && ln -s libsodium.so.23.3.0 libsodium.so.23; }; })
libtool: install: (cd /usr/lib && { ln -s -f libsodium.so.23.3.0 libsodium.so || { rm -f libsodium.so && ln -s libsodium.so.23.3.0 libsodium.so; }; })
libtool: install: /bin/install -c .libs/libsodium.lai /usr/lib/libsodium.la
libtool: install: /bin/install -c .libs/libsodium.a /usr/lib/libsodium.a
libtool: install: chmod 644 /usr/lib/libsodium.a
libtool: install: ranlib /usr/lib/libsodium.a
libtool: finish: PATH="/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin:/root/bin:/sbin" ldconfig -n /usr/lib
----------------------------------------------------------------------
Libraries have been installed in:
   /usr/lib

If you ever happen to want to link against installed libraries
in a given directory, LIBDIR, you must either use libtool, and
specify the full pathname of the library, or use the '-LLIBDIR'
flag during linking and do at least one of the following:
   - add LIBDIR to the 'LD_LIBRARY_PATH' environment variable
     during execution
   - add LIBDIR to the 'LD_RUN_PATH' environment variable
     during linking
   - use the '-Wl,-rpath -Wl,LIBDIR' linker flag
   - have your system administrator add LIBDIR to '/etc/ld.so.conf'

See any operating system documentation about shared libraries for
more information, such as the ld(1) and ld.so(8) manual pages.
----------------------------------------------------------------------
make[4]: Nothing to be done for `install-data-am'.
make[4]: Leaving directory `/home/ec2-user/libsodium-1.0.18/src/libsodium'
make[3]: Leaving directory `/home/ec2-user/libsodium-1.0.18/src/libsodium'
make[2]: Leaving directory `/home/ec2-user/libsodium-1.0.18/src/libsodium'
make[2]: Entering directory `/home/ec2-user/libsodium-1.0.18/src'
make[3]: Entering directory `/home/ec2-user/libsodium-1.0.18/src'
make[3]: Nothing to be done for `install-exec-am'.
make[3]: Nothing to be done for `install-data-am'.
make[3]: Leaving directory `/home/ec2-user/libsodium-1.0.18/src'
make[2]: Leaving directory `/home/ec2-user/libsodium-1.0.18/src'
make[1]: Leaving directory `/home/ec2-user/libsodium-1.0.18/src'
Making install in test
make[1]: Entering directory `/home/ec2-user/libsodium-1.0.18/test'
Making install in default
make[2]: Entering directory `/home/ec2-user/libsodium-1.0.18/test/default'
make[3]: Entering directory `/home/ec2-user/libsodium-1.0.18/test/default'
make[3]: Nothing to be done for `install-exec-am'.
make[3]: Nothing to be done for `install-data-am'.
make[3]: Leaving directory `/home/ec2-user/libsodium-1.0.18/test/default'
make[2]: Leaving directory `/home/ec2-user/libsodium-1.0.18/test/default'
make[2]: Entering directory `/home/ec2-user/libsodium-1.0.18/test'
make[3]: Entering directory `/home/ec2-user/libsodium-1.0.18/test'
make[3]: Nothing to be done for `install-exec-am'.
make[3]: Nothing to be done for `install-data-am'.
make[3]: Leaving directory `/home/ec2-user/libsodium-1.0.18/test'
make[2]: Leaving directory `/home/ec2-user/libsodium-1.0.18/test'
make[1]: Leaving directory `/home/ec2-user/libsodium-1.0.18/test'
make[1]: Entering directory `/home/ec2-user/libsodium-1.0.18'
make[2]: Entering directory `/home/ec2-user/libsodium-1.0.18'
make[2]: Nothing to be done for `install-exec-am'.
 /bin/mkdir -p '/usr/lib/pkgconfig'
 /bin/install -c -m 644 libsodium.pc '/usr/lib/pkgconfig'
make[2]: Leaving directory `/home/ec2-user/libsodium-1.0.18'
make[1]: Leaving directory `/home/ec2-user/libsodium-1.0.18'
running install
running bdist_egg
running egg_info
creating shadowsocks.egg-info
writing shadowsocks.egg-info/PKG-INFO
writing top-level names to shadowsocks.egg-info/top_level.txt
writing dependency_links to shadowsocks.egg-info/dependency_links.txt
writing entry points to shadowsocks.egg-info/entry_points.txt
writing manifest file 'shadowsocks.egg-info/SOURCES.txt'
reading manifest file 'shadowsocks.egg-info/SOURCES.txt'
reading manifest template 'MANIFEST.in'
writing manifest file 'shadowsocks.egg-info/SOURCES.txt'
installing library code to build/bdist.linux-x86_64/egg
running install_lib
running build_py
creating build
creating build/lib
creating build/lib/shadowsocks
copying shadowsocks/__init__.py -> build/lib/shadowsocks
copying shadowsocks/asyncdns.py -> build/lib/shadowsocks
copying shadowsocks/common.py -> build/lib/shadowsocks
copying shadowsocks/cryptor.py -> build/lib/shadowsocks
copying shadowsocks/daemon.py -> build/lib/shadowsocks
copying shadowsocks/eventloop.py -> build/lib/shadowsocks
copying shadowsocks/local.py -> build/lib/shadowsocks
copying shadowsocks/lru_cache.py -> build/lib/shadowsocks
copying shadowsocks/manager.py -> build/lib/shadowsocks
copying shadowsocks/server.py -> build/lib/shadowsocks
copying shadowsocks/shell.py -> build/lib/shadowsocks
copying shadowsocks/tcprelay.py -> build/lib/shadowsocks
copying shadowsocks/tunnel.py -> build/lib/shadowsocks
copying shadowsocks/udprelay.py -> build/lib/shadowsocks
creating build/lib/shadowsocks/crypto
copying shadowsocks/crypto/__init__.py -> build/lib/shadowsocks/crypto
copying shadowsocks/crypto/aead.py -> build/lib/shadowsocks/crypto
copying shadowsocks/crypto/hkdf.py -> build/lib/shadowsocks/crypto
copying shadowsocks/crypto/mbedtls.py -> build/lib/shadowsocks/crypto
copying shadowsocks/crypto/openssl.py -> build/lib/shadowsocks/crypto
copying shadowsocks/crypto/rc4_md5.py -> build/lib/shadowsocks/crypto
copying shadowsocks/crypto/sodium.py -> build/lib/shadowsocks/crypto
copying shadowsocks/crypto/table.py -> build/lib/shadowsocks/crypto
copying shadowsocks/crypto/util.py -> build/lib/shadowsocks/crypto
creating build/bdist.linux-x86_64
creating build/bdist.linux-x86_64/egg
creating build/bdist.linux-x86_64/egg/shadowsocks
copying build/lib/shadowsocks/__init__.py -> build/bdist.linux-x86_64/egg/shadowsocks
copying build/lib/shadowsocks/asyncdns.py -> build/bdist.linux-x86_64/egg/shadowsocks
copying build/lib/shadowsocks/common.py -> build/bdist.linux-x86_64/egg/shadowsocks
copying build/lib/shadowsocks/cryptor.py -> build/bdist.linux-x86_64/egg/shadowsocks
copying build/lib/shadowsocks/daemon.py -> build/bdist.linux-x86_64/egg/shadowsocks
copying build/lib/shadowsocks/eventloop.py -> build/bdist.linux-x86_64/egg/shadowsocks
copying build/lib/shadowsocks/local.py -> build/bdist.linux-x86_64/egg/shadowsocks
copying build/lib/shadowsocks/lru_cache.py -> build/bdist.linux-x86_64/egg/shadowsocks
copying build/lib/shadowsocks/manager.py -> build/bdist.linux-x86_64/egg/shadowsocks
copying build/lib/shadowsocks/server.py -> build/bdist.linux-x86_64/egg/shadowsocks
copying build/lib/shadowsocks/shell.py -> build/bdist.linux-x86_64/egg/shadowsocks
copying build/lib/shadowsocks/tcprelay.py -> build/bdist.linux-x86_64/egg/shadowsocks
copying build/lib/shadowsocks/tunnel.py -> build/bdist.linux-x86_64/egg/shadowsocks
copying build/lib/shadowsocks/udprelay.py -> build/bdist.linux-x86_64/egg/shadowsocks
creating build/bdist.linux-x86_64/egg/shadowsocks/crypto
copying build/lib/shadowsocks/crypto/__init__.py -> build/bdist.linux-x86_64/egg/shadowsocks/crypto
copying build/lib/shadowsocks/crypto/aead.py -> build/bdist.linux-x86_64/egg/shadowsocks/crypto
copying build/lib/shadowsocks/crypto/hkdf.py -> build/bdist.linux-x86_64/egg/shadowsocks/crypto
copying build/lib/shadowsocks/crypto/mbedtls.py -> build/bdist.linux-x86_64/egg/shadowsocks/crypto
copying build/lib/shadowsocks/crypto/openssl.py -> build/bdist.linux-x86_64/egg/shadowsocks/crypto
copying build/lib/shadowsocks/crypto/rc4_md5.py -> build/bdist.linux-x86_64/egg/shadowsocks/crypto
copying build/lib/shadowsocks/crypto/sodium.py -> build/bdist.linux-x86_64/egg/shadowsocks/crypto
copying build/lib/shadowsocks/crypto/table.py -> build/bdist.linux-x86_64/egg/shadowsocks/crypto
copying build/lib/shadowsocks/crypto/util.py -> build/bdist.linux-x86_64/egg/shadowsocks/crypto
byte-compiling build/bdist.linux-x86_64/egg/shadowsocks/__init__.py to __init__.pyc
byte-compiling build/bdist.linux-x86_64/egg/shadowsocks/asyncdns.py to asyncdns.pyc
byte-compiling build/bdist.linux-x86_64/egg/shadowsocks/common.py to common.pyc
byte-compiling build/bdist.linux-x86_64/egg/shadowsocks/cryptor.py to cryptor.pyc
byte-compiling build/bdist.linux-x86_64/egg/shadowsocks/daemon.py to daemon.pyc
byte-compiling build/bdist.linux-x86_64/egg/shadowsocks/eventloop.py to eventloop.pyc
byte-compiling build/bdist.linux-x86_64/egg/shadowsocks/local.py to local.pyc
byte-compiling build/bdist.linux-x86_64/egg/shadowsocks/lru_cache.py to lru_cache.pyc
byte-compiling build/bdist.linux-x86_64/egg/shadowsocks/manager.py to manager.pyc
byte-compiling build/bdist.linux-x86_64/egg/shadowsocks/server.py to server.pyc
byte-compiling build/bdist.linux-x86_64/egg/shadowsocks/shell.py to shell.pyc
byte-compiling build/bdist.linux-x86_64/egg/shadowsocks/tcprelay.py to tcprelay.pyc
byte-compiling build/bdist.linux-x86_64/egg/shadowsocks/tunnel.py to tunnel.pyc
byte-compiling build/bdist.linux-x86_64/egg/shadowsocks/udprelay.py to udprelay.pyc
byte-compiling build/bdist.linux-x86_64/egg/shadowsocks/crypto/__init__.py to __init__.pyc
byte-compiling build/bdist.linux-x86_64/egg/shadowsocks/crypto/aead.py to aead.pyc
byte-compiling build/bdist.linux-x86_64/egg/shadowsocks/crypto/hkdf.py to hkdf.pyc
byte-compiling build/bdist.linux-x86_64/egg/shadowsocks/crypto/mbedtls.py to mbedtls.pyc
byte-compiling build/bdist.linux-x86_64/egg/shadowsocks/crypto/openssl.py to openssl.pyc
byte-compiling build/bdist.linux-x86_64/egg/shadowsocks/crypto/rc4_md5.py to rc4_md5.pyc
byte-compiling build/bdist.linux-x86_64/egg/shadowsocks/crypto/sodium.py to sodium.pyc
byte-compiling build/bdist.linux-x86_64/egg/shadowsocks/crypto/table.py to table.pyc
byte-compiling build/bdist.linux-x86_64/egg/shadowsocks/crypto/util.py to util.pyc
creating build/bdist.linux-x86_64/egg/EGG-INFO
copying shadowsocks.egg-info/PKG-INFO -> build/bdist.linux-x86_64/egg/EGG-INFO
copying shadowsocks.egg-info/SOURCES.txt -> build/bdist.linux-x86_64/egg/EGG-INFO
copying shadowsocks.egg-info/dependency_links.txt -> build/bdist.linux-x86_64/egg/EGG-INFO
copying shadowsocks.egg-info/entry_points.txt -> build/bdist.linux-x86_64/egg/EGG-INFO
copying shadowsocks.egg-info/top_level.txt -> build/bdist.linux-x86_64/egg/EGG-INFO
creating dist
creating 'dist/shadowsocks-3.0.0-py2.7.egg' and adding 'build/bdist.linux-x86_64/egg' to it
removing 'build/bdist.linux-x86_64/egg' (and everything under it)
Processing shadowsocks-3.0.0-py2.7.egg
creating /usr/lib/python2.7/site-packages/shadowsocks-3.0.0-py2.7.egg
Extracting shadowsocks-3.0.0-py2.7.egg to /usr/lib/python2.7/site-packages
Adding shadowsocks 3.0.0 to easy-install.pth file
Installing sslocal script to /usr/bin
Installing ssserver script to /usr/bin

Installed /usr/lib/python2.7/site-packages/shadowsocks-3.0.0-py2.7.egg
Processing dependencies for shadowsocks==3.0.0
Finished processing dependencies for shadowsocks==3.0.0
writing list of installed files to '/usr/local/shadowsocks_python.log'
Starting Shadowsocks success

Congratulations, Shadowsocks-Python server install completed!
Your Server IP        :  3.88.156.118 
Your Server Port      :  5000 
Your Password         :  XuYijie312416 
Your Encryption Method:  aes-256-gcm 

Your QR Code: (For Shadowsocks Windows, OSX, Android and iOS clients)
 ss://YWVzLTI1Ni1nY206WHVZaWppZTMxMjQxNkAzLjg4LjE1Ni4xMTg6NTAwMA== 
Your QR Code has been saved as a PNG file path:
 /home/ec2-user/shadowsocks_python_qr.png 

Welcome to visit: https://teddysun.com/486.html
Enjoy it!

[root@ip-172-31-29-157 ec2-user]



```


---

#  三、连接方法

下载小飞机，手机版小飞机为例


![在这里插入图片描述](https://img-blog.csdnimg.cn/d2a3e9b457224d09a535a6e889b54ff0.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA57mB5Y2O5bC95aS05ruh5piv5q6H,size_20,color_FFFFFF,t_70,g_se,x_16)



# 总结
