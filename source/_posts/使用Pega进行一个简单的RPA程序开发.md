---
title: 使用Pega进行一个简单的RPA程序开发
date: 2022-06-24 15:22:34
categories: Pega
tags:
    - Pega
    - RPA
cover: https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/PegaLogo.jpg
---
# 前言

Pega 和 RPA 可能都是大多数程序员没有了解过的东西，本人在一家外企做 BPM 和 RPA 项目的部门工作，进入公司的时候就遇到了稀缺技术的学习资源少的问题，因此在此将我学习和工作积累下来的一些知识记录下来。

`放个 Pega BPM 的笔记连接`
[CSDN](https://blog.csdn.net/qq_48922459/article/details/126729159?spm=1001.2014.3001.5501)
[Github仓库](https://github.com/XuYijie000416/Pega)

---


# 一、Pega是什么
Pega总的来说可以做两件事——BPM(Business Process Management) 和 RPA(Robotic Process Automation)。关于它的最好的事情之一是开发没有代码开发的应用程序，其中整个开发过程都是可视化的。通过这种方式，您可以构建可扩展的自动化应用程序，并从应用程序中引入许多其他必要的功能。当然，如果你喜欢代码，也可以在 Pega 中方便地嵌入 Java 或者 C# 代码。
##  BPM（业务流程管理）
大型企业的业务流程非常复杂，并且需要高效和自动化的重复性操作，比如一个企业的ERP系统有很复杂的逻辑，这用代码实现起来就非常的复杂，并且后来的程序员也很难读懂这些流程，Pega就是为此而生的，它可以将业务流程以可视化的方式进行构建和展示出来，就和画流程图一样，开发方便、后来者读懂流程也更容易，这就叫BPM。
##  RPA（机器人流程自动化）
比如一个企业的电子对账系统上有很多重复性工作，需要会计不断地点击，对比数据，确定账单等，这些工作就可以使用 RPA 来完成，将这个工作的操作方法以可视化的方式告诉Pega，Pega将在设定的时间自动打开电子对账系统，进行对账操作，并通过邮件告诉相关会计人员对账结果。

----

# 二、构建一个简单的 RPA 程序
我做一个简单的例子，要实现自动打开浏览器，进入Bing主页，然后搜索上海天气，选中一条搜索结果，将里面的天气信息输出来。

## 新建一个Pega项目
打开Visual Studio，当然是配置好Pega的VS，配置教程暂时不写，我也还没弄清楚。

![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/PegaRPA0.png)


## 新建universal web application
指定一个程序要进行操作的网站
右键新建的Pega项目“Project6”，添加-新建项，选择universal web application，不选web application是因为它仅支持IE浏览器。

![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/PegaRPA1.png)
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/PegaRPA2.png)
##  抓取页面元素
双击右边我们新建的universal web，会打开中间界面，单击左边的web application，右下角会出现属性设置。

![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/PegaRPA3.png)
startPage是要打开的网站的网址，我们用Bing搜索引擎来演示。

![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/PegaRPA4.png)

设置完成后点击start interrogation，会打开刚刚设置的startPage网址，并出现一个小弹窗。

![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/PegaRPA5.png)
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/PegaRPA6.png)
我们一般选择模式为select elements，拖动弹窗的那个靶心一样的图标，放到要抓取的页面元素上
，这里我抓取搜索框。

![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/PegaRPA7.png)
会出现抓取的页面结构，我们选择 input 标签，点击 create

![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/PegaRPA8.png)
VS中就出现了我们刚刚抓取的搜索框

![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/PegaRPA9.png)

按照刚刚的方法，抓取搜索框右边的搜索按钮


----

# 三、构建流程
##  新建Automation
新建的 Pega 项目应该自带一个 Automation1，如果没有，就自己新建一个，还是右键 pega 项目，新建项

![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/PegaRPA10.png)

##  开始构建
流程的起始要添加一个 Execute ，然后点击左边的 universal web application，选择属性栏里面的 startPage，设置打开的网址，然后选择动作栏里面的 started，然后等待这个动作，防止电脑卡，浏览器打开的慢导致报错，然后选择方法栏里面的 start 方法，setup连接 start，选中我门抓取的搜索框，在左下方属性栏把 text 属性拖进来，连接到fried。图中蓝色的线代表传值，黄色的线代表流程走向。工具箱里面的 string 方法是创建一个变量，赋值给搜索框的 text 属性，然后选中我们抓取的搜索按钮 searchButton，在方法栏中把performClick拖出来，代表点击一下搜索按钮。



[video(video-4OWXf9Z4-1656057397844)(type-csdn)(url-https://live.csdn.net/v/embed/218751)(image-https://video-community.csdnimg.cn/vod-84deb4/faa34210cf1041f48c148f54c8acad74/snapshots/946076a9409845c8b503d99022716a25-00005.jpg?auth_key=4809656909-0-0-dc9e0ca767056dbe4eb6437e7e61053c)(title-Pega RPA流程构建视频)]




上面的演示已经演示了80%的构建过程，下面是完整的，最后的三步要你们自己研究哦，成功后，会和下面的成果演示一样
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/PegaRPA11.png)


---

#  成果演示
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/PegaRPA12.gif)


