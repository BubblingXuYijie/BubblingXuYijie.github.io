---
title: 用纯CSS改变网页滚动条的样式
date: 2020-06-29 19:07:58
categories: Web前端
tags:
    - HTML
    - CSS
cover: https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/HtmlCssLogo.jpg
---
## 代码
```html
<style>
    .test{
        width: 50px;
        height: 200px;
        overflow: auto;
        float: left;
        margin: 5px;
        border: none;
    }
    .scrollbar{
        width: 30px;
        height: 300px;
        margin: 0 auto;
    }
    /*滚动条整体样式*/
    .test-1::-webkit-scrollbar{
        /*高度宽度对应横竖滚动条的尺寸*/
        width: 30px;
        height: 5px;
    }
    /*滚动条里面的滚动的方块*/
    .test-1::-webkit-scrollbar-thumb{
        border-radius: 10px;
        -webkit-box-shadow: inset 0 0 5px rgba(0,0,0,0.2);
        background-color: #535353;
    }
    /*滚动条的轨道*/
    .test-1::-webkit-scrollbar-track{
        -webkit-box-shadow: inset 0 0 5px rgba(0,0,0,0.2);
        border-radius: 10px;
        background-color: #EDEDED;
    }
</style>
<body class="test test-1 scrollbar">
    <div style="background-color: red;height: 2500px;">
    </div>
</body>
```
## 如图所示滚动条的样式已改变
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/CSS滚动条.png)
