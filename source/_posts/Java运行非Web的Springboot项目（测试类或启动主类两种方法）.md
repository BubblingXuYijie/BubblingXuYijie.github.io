---
title: Java运行非Web的Springboot项目（测试类或启动主类两种方法）
date: 2021-12-05 14:04:19
categories: Java
cover: https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/springbootLogo.jpeg
---
# 前言

<font color=#999AAA >如果springboot不是一个Web项目，大家知道，项目启动以后马上就会停止，并且 controller 等各层里面的方法也不会被执行，下面有两种方式可以运行容器里面的方法，测试类或者修改启动主类，都非常简单，几行代码的事情。</font>

<hr style=" border:solid; width:100px; height:1px;" color=#000000 size=1">


# 一、创建打开一个Springboot项目


<font color=#999AAA >使用idea，选择Spring Initializr进行创建

![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/启动非webSprigboot0.png)


<font color=#999AAA >next，这一步不要勾选 Spring Web 依赖，不然就是个 Web 项目了


![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/启动非webSprigboot1.png)

<font color=#999AAA >完成以后大家发现，已经为我们创建好了 测试类，那我们就先讲用测试类运行吧
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/启动非webSprigboot2.png)

<font color=#999AAA >先写好基本结构，我的这个 demo方法就是一个 System.out.println("Hello World");

![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/启动非webSprigboot3.png)
<hr style=" border:solid; width:100px; height:1px;" color=#000000 size=1">

# 二、运行项目


##  1、Test测试类运行
<font color=#999AAA >打开测试类，我的叫 UseToTestApplicationTests，不同项目名字不一样，里面的初始代码是这样的，每个人都一样

![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/启动非webSprigboot4.png)


<font color=#999AAA >在里面直接 @Autowired 你的 controller 层的文件，在下面 contextLoads 里面调用方法，右键，运行 contextLoads 就可以了。然后输出了 HelloWorld

![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/启动非webSprigboot5.png)
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/启动非webSprigboot6.png)

##  2、启动主类运行

<font color=#999AAA >下面使用 UseToTestApplication 启动主类来运行项目

<font color=#999AAA >修改代码为下图所示，要获取哪个容器就`getBean`哪个容器名，右键启动 UseToTestApplication

```java
ConfigurableApplicationContext context = SpringApplication.run(UseToTestApplication.class, args);
        //获取容器DemoController
        DemoController demoController = (DemoController) context.getBean("demoController");
        //DemoController获取成功，调用demo方法
        demoController.demo();
```
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/启动非webSprigboot7.png)

<font color=#999AAA >输出 HelloWorld

![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/启动非webSprigboot8.png)

<hr style=" border:solid; width:100px; height:1px;" color=#000000 size=1">


# 三、持续运行不停止（定时任务、自动执行）

<font color=#999AAA >如果想要让项目不停止，一直打印HelloWorld，可以在启动类 UseToTestApplication  上添加注解 `@EnableScheduling` ，意思为开启定时任务，这个时候启动类就不能修改成上面的`getBean`那样了，要改回原来的样子

<font color=#999AAA >然后在 controller 层的DemoControlelr里面加上注解 `@Component` 可以确保这个类会被定时任务扫描到，然后在下面的 demo 方法上加上 `@Scheduled(fixedRate = 3000)`，意思为每1000毫秒执行一次 demo 方法


![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/启动非webSprigboot9.png)
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/启动非webSprigboot10.png)


<font color=#999AAA >启动主类，运行结果，每1秒打印一个 HelloWorld ，永不停止
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/启动非webSprigboot11.png)

<hr style=" border:solid; width:100px; height:1px;" color=#000000 size=1">

# 总结
<font color=#999AAA >如果需要更详细的定时任务的操作，我的另一篇文章有详解，可以定制逻辑更复杂的定时任务

传送门：[Java实现非Web项目的Springboot定时任务（每3秒自动执行一次）](https://blog.csdn.net/qq_48922459/article/details/121687993?spm=1001.2014.3001.5501)
