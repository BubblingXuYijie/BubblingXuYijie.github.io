---
title: 使用Wireshark抓取用户在网站的登陆密码
date: 2020-07-04 16:31:19
categories: 网络
tags:
    - Wireshark
    - 网络
cover: https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/WireShark.jpg
sticky: 2
---
# 下载Wireshark
[Wireshark下载地址](https://www.wireshark.org/#download)
下载后傻瓜式安装

![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/WireShark0.png)
# 获取想要抓取网站的IP

用我们学校的教务系统为例子

![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/WireShark1.png)

记下网站：jw1.wzbc.edu.cn（注意哦，要去掉http和/，比如http://www.4399.com/，我们只需要记下4399.com即可）

win+R 打开“运行”，在框里输入“cmd”，确定

![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/WireShark2.png)

然后在刚刚打开的黑色会话框里输入：ping jw1.wzbc.edu.cn ,然后回车，得到的IP为10.151.160.43，记下这个IP

![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/WireShark3.png)

# 开始抓包
打开Wireshark，选择你使用的网络，如果是WIFI就双击“WIFI”，有线就选择“以太网”

![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/WireShark4.png)

我是有线，所以我双击“以太网”打开，是这个样子，上面的数据先不用管，我们先点击左上角红色的方块停止抓包
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/WireShark5.png)

然后在这里输入http and ip.addr==10.151.160.43（这句话是筛选数据，就是只显示IP为我们要抓取网站的数据） ，点击右边的蓝色箭头
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/WireShark6.png)

点击Wireshark左上角蓝色的鲨鱼鳍，开始抓包
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/WireShark7.png)

之后，打开我们的教务系统登陆页面，登录账户![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/WireShark8.png)
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/WireShark9.png)

登陆成功后，查看Wireshark,我们发现有一个POST（POST请求是浏览器向服务器提交数据的方式，我们的用户名和密码就是这样提交出去的），单击点开，发现我们的账号和密码，就被抓取到了，在HTML Form Encoded里面显示出来了
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/WireShark10.png)

# 结束
这就是Wireshark简单的抓取登录密码的教程，只适用于http协议的网站

