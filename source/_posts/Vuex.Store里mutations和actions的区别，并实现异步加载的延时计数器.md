---
title: Vuex.Store里mutations和actions的区别，并实现异步加载的延时计数器
date: 2021-05-10 11:55:11
categories: Web前端
tags: 
    - Vue
cover: /img/Vue.jpeg
---
# 前言

mutations和actions都是Vuex.Store里定义函数的方法

mutations定义的函数的参数都有一个state，表示store里的整个state数据，同步加载

actions定义的函数里的参数是content，代表整个store对象，用于异步加载


# 先上代码在说话
便于比较，我把mutations和actions写到了一起

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>store-mutations和actions实现的计数器</title>
    <script src="vue.js"></script>
    <script src="vuex.js"></script>
</head>
<body>
    <div id="app">
        <button @click="add">mutations</button>
        <button @click="add1">actions</button>
        <p>{{count}}</p>
    </div>

    <script>
        var store = new Vuex.Store({
            state:{
                count:0
            },
            //mutations定义的函数的参数都是state，表示整个state数据，同步加载
            mutations:{
                add:function(state,value){
                    state.count++,
                    console.log(state);
                    console.log(value);
                }
            },
            //actions定义的函数使用dispatch触发，函数里的content是整个store对象，异步加载
            actions:{
                add:function(content){
                    console.log(content);
                    
                    this.timer = setTimeout(()=>{   //设置延迟执行
                        content.commit("add")
                    },1000);
                }
            }
        })
        var a = new Vue({
            el:"#app",
            store,
            data:{
                pare:"五一劳动节"
            },
            methods: {
                add:function(){
                    //pare可以传递到mutations的add方法里，在控制台输出pare
                    this.$store.commit("add",this.pare)
                },
                add1:function(){
                    this.$store.dispatch("add")
                }
            },
            computed:Vuex.mapState({
                count:state => state.count
            })
        })
    </script>
</body>
</html>
```


# 解析
DOM结构是这样的

```html
<div id="app">
        <button @click="add">mutations</button>
        <button @click="add1">actions</button>
        <p>{{count}}</p>
    </div>
```
两个按钮分别调用了Vue里的add和add1方法，add和add1的区别在于commit和dispatch

我还在commit里面加上了一个参数pare，这个pare是定义在Vue的data里的，值是“五一劳动节”，这个值可以传递到mutations里的add方法里，下面会介绍

```javascript
 methods: {
        add:function(){
             //pare可以传递到mutations的add方法里，在控制台输出pare
             this.$store.commit("add",this.pare)
         },
         add1:function(){
             this.$store.dispatch("add")
         }
     },
```
commit调用的是mutations里的add方法，dispatch调用的是actions里的add方法

```javascript
mutations:{
         add:function(state,value){
             state.count++,
             console.log(state);
             console.log(value);
         }
     },
     //actions定义的函数使用dispatch触发，函数里的content是整个store对象，异步加载
actions:{
	    add:function(content){
	        console.log(content);
	        
	        this.timer = setTimeout(()=>{   //设置延迟执行
	            content.commit("add")
	        },1000);
 		}
	}
```
#  运行代码
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210510113954880.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ4OTIyNDU5,size_16,color_FFFFFF,t_70)

如上图，我们先点击mutations按钮，右边控制台输出了mutations里add方法的state的值和传来的pare的值

接下来我们再点击actions按钮，右边控制台输出了actions里add方法的content的值，content的值里面包含上面的state
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210510114826664.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ4OTIyNDU5,size_16,color_FFFFFF,t_70)



# 异步加载延时计数器
看这一段代码，我们在actions的add方法里面，并没有直接使用count++来使计数加1，而是调用了mutations里的add方法来进行加1，目的就是实现异步加载，把commit("add")放在setTiomeout里面，代表1000毫秒以后调用mutations里的add方法

```javascript
mutations:{
         add:function(state,value){
             state.count++,
             console.log(state);
             console.log(value);
         }
     },
     //actions定义的函数使用dispatch触发，函数里的content是整个store对象，异步加载
actions:{
	    add:function(content){
	        console.log(content);
	        
	        this.timer = setTimeout(()=>{   //设置延迟执行
	            content.commit("add")
	        },1000);
 		}
	}
```

当然你也可以直接写成

```javascript
this.timer = setTimeout(()=>{   //设置延迟执行
               content.state.count++
          },1000);
     }
```

