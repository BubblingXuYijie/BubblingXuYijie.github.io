---
title: Vue动态路由的Query和Params传参方式
date: 2021-05-10 11:16:47
categories: Web前端
tags:
    - Vue
cover: /img/Vue.jpeg
---
# 一、Query传参

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>动态路由Query方式</title>
    <script src="vue.js"></script>
    <script src="vue-router.js"></script>
</head>
<body>
    <div id="app">
        <router-link tag="div" to="/login?id=1111&name=2222">登录</router-link>
        <router-view></router-view>
    </div>
    <template id="lg">
        <div>
            {{this.$route.query.id}}
        </div>
    </template>
    <script>
        var Login = {
            template:"#lg",
        }
        var router = new VueRouter({
            routes: [
                {path:"/login",component:Login}
            ]
        })
        var a = new Vue({
            el:"#app",
            router
        })
    </script>
</body>
</html>
```

# 二、Params传参

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>动态路由params方法</title>
    <script src="vue.js"></script>
    <script src="vue-router.js"></script>
</head>
<body>
    <div id="app">
        <router-link tag="div" to="/login/1111/2222">登录</router-link>
        <router-view></router-view>
    </div>
    <template id="lg">
        <div>
            {{this.$route.params.id}}{{this.$route.params.name}}
        </div>
    </template>
    <script>
        var Login = {
            template:"#lg",
        }
        var router = new VueRouter({
            routes: [
                {path:"/login/:name/:id",component:Login}
            ]
        })
        var a = new Vue({
            el:"#app",
            router
        })
    </script>
</body>
</html>
```


# 两种传参方式的区别
##  Query
Query方法的router-link里面的to，to="/login?id=1111&name=2222"，
路由配置是

```html
var router = new VueRouter({
            routes: [
                {path:"/login",component:Login}
            ]
        })
```
Query方式的网页地址栏后面是这样：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210510110403931.png)
##  Params


Params方法的router-link里面的to，to="/login/1111/2222"，
路由配置是

```html
 var router = new VueRouter({
            routes: [
                {path:"/login/:name/:id",component:Login}
            ]
        })
```
Params方式的网页地址栏后面是这样：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210510110420216.png)
注意Params传参的路由配置里，我故意把path里写成{path:"/login/:name/:id",component:Login}，把name放到了前面，所以name排在第一位，对应router-link里面的to="/login/1111/2222"的第一位“1111”

```html
{{this.$route.params.id}}{{this.$route.params.name}}
```
这两句获取参数是下面这样的，也就是id是2222，name是1111
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210510111041254.png)
