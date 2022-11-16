---
title: 用纯CSS改变网页滚动条的样式
date: 2020-06-29 19:26:55
categories: Web前端
tags:
    - HTML
    - CSS
cover: /img/HtmlCssLogo.jpg
---
## 代码

```html
<span class="name">Name</span>
<style>
.name{   
    font-size:50px;
    background-clip: border-box;
    -webkit-background-clip: text;
    -webkit-text-fill-color: transparent;
    color: #FF512F;
    font-weight: 700;
    text-shadow: 0px 0px 7px #ffd800;
    background-image: linear-gradient(90deg, #ffd800 0%, #ff512f 100%, #fff);
    animation: glow-animation 2s infinite linear;
    color: #FFC0CB;
    box-sizing: border-box;
    vertical-align: inherit;
}
@keyframes glow-animation{
    0%{filter:hue-rotate(-360deg)}
    100%{filter:hue-rotate(360deg)}
}
</style>
```
## 实现效果
实际动画很流畅，下图为录制的GIF帧率较低
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020062919262875.gif)
