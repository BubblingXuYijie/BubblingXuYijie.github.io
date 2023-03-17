---
title: Vue使用alibaba的iconfont矢量图标
date: 2021-11-30 20:35:11
categories: Web前端
tags:
    - Vue
cover: https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/alibabaIcon0.png
---
####  网上有很多引入方法，官方也给出了三种引入方法，但是大多数人引入后都不会显示，或者是不能自定义样式，下面这种方法不会存在上面的那些问题，是目前最好的引入方式。
<hr>


1、网址 [阿里巴巴矢量图库](https://www.iconfont.cn/) ，要求登陆后使用，先注册登录吧

2、登陆后，搜索想要的图标，比如用户、购物车什么的
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/alibabaIcon0.png)
<hr>

3、鼠标移动到想要的图标上，点击“添加入库”，建议不要直接用复制SVG代码引入或者其他引入方式，先加入库在下载下来是最好的选择。
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/alibabaIcon1.png)
<hr>

4、打开网址右上角的购物车，将所选的图标“添加至项目”
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/alibabaIcon2.png)
<hr>

5、下载至本地，会得到一个压缩包，解压到当前文件夹

![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/alibabaIcon3.png)

<hr>

6、会出现这些内容，将除了demo开头的所有文件全部复制进项目的“asset”文件夹下的“iconfont”文件夹，没有的话自己新建文件夹![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/alibabaIcon4.png)
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/alibabaIcon5.png)
<hr>

7、接下来用浏览器打开解压出来的文件夹里的demo_index.html，这些代码就是对应的每个图标，等下要用到
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/alibabaIcon6.png)
<hr>

8、在代码中这样写，你的项目网页就会显示第一个图标，上图== &#xe601 ==对应的那个图标

```html
<template>

 <span class="iconfont">&#xe601;</span>
<template>

<script>
    import '../asset/iconfont/iconfont.css'; //这里是你的iconfont.css路径，和我的可能不一
</script>
```

<hr>

9、要修改图标样式，就在style标签里修改.iconfont类就可以了

```html
<template>

 <span class="iconfont">&#xe601;</span>
<template>

<script>
    import '../asset/iconfont/iconfont.css'; //这里是你的iconfont.css路径，和我的可能不一
</script>


<style>
.iconfont {
  font-family: "iconfont" !important;
  font-size: 16px;
  font-style: normal;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}
</style>
```

<hr>

10、结束，在哪个页面使用图标就在哪个页面先 `import '../asset/iconfont/iconfont.css';` 一下就可以了

11、==注意==，如果还想从阿里矢量图库新添加一些图标，则需要删除项目里的iconfont文件夹，重新从本文章第2步开始，选择，下载，引入
