---
title: 解决 Vue 使用 $ref 调用子组件方法时的控制台报错
date: 2022-02-16 21:59:29
categories: Web前端
tags:
    - Vue
cover: https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/Vue.jpeg
---
# 报错原因

<font color=#999AAA >代码和控制台报错, Uncaught TypeError: Cannot read properties of undefined</font>

```javascript
this.$refs.chatting.getMessageLib()
```

![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/VueRef报错0.png)

<font color=#999AAA >上述报错大多出现在操作弹窗子组件的页面元素的情景中，原因是子组件还未渲染到父组件的 `DOM` 中，就开始对子组件的元素进行操作了</font>

<hr style=" border:solid; width:100px; height:1px;" color=#000000 size=1">



# 解决办法


<font color=#999AAA >解决办法是把 `ref` 方法放到 vue 的内置函数中，`$nextTick` 可以等待 DOM 渲染完成后在调用里面的方法，这样控制台就不会出现报错了。

```javascript
this.$nextTick(() => {
    this.$refs.chatting.getMessageLib()
})
```

<hr style=" border:solid; width:100px; height:1px;" color=#000000 size=1">

