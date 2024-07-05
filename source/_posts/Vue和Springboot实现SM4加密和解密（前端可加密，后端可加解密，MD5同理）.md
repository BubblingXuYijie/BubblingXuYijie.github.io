---
title: Vue和Springboot实现SM4加密和解密（前、后端均可）
date: 2021-12-24 17:00:05
categories: Java
tags:
    - Vue
    - SpringBoot
    - SM4
    - 数据加密
cover: https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/springbootLogo.jpeg
---

# 前言

<font color=#999AAA >网站配置 https 比较麻烦，所以为了我们的用户账户安全，密码在从前端传输到后端的过程中，最好加密一下，选用 SM4 有两个原因，一是国产加密算法，二是这个国密算法是对称的，只要加密和解密的 key 和 vi 相同，可以很容易的解密，同时需要匹配 key 和 vi 又兼顾了安全。</font>

<font color=#999AAA >我下面会提供前端的 SM4 加密 js 文件，vue 项目也可以使用，还有 Java 的 SM4 加密和解密文件。可实现前端加密传输到后端解密，存到数据库，后端也可以解密传输到前端进行明文的显示。</font>


`加密源代码网上有很多，但是代码语法和jar包陈旧，导致新版本jdk无法运行；以及 js 使用的语法太旧，导致 Vue 编译不通过（即使不使用 ESLint也不通过），所以我这个在他们的基础上修改了，后端只需引入一个依赖，前端语法已经规范修改，而且可通过 ESLint 的检测。`

---

> 另外前后端的 SM4 加解密我已经上传到 npm 和 maven 中央仓库了，你们可以 npm install sm4util 和 引入到 pom 使用
```xml
<!--引入-->
<dependency>
    <groupId>icu.xuyijie</groupId>
    <artifactId>SM4Utils</artifactId>
    <version>1.4.8</version>
</dependency>
```

```bash
// 安装
npm install sm4util
```
```js
//引入和使用
import {SM4Util} from "sm4util";
const sm4 = new SM4Util();
sm4.encryptDefault_ECB('123456');
```

```java
// ECB 加密模式
//不使用自定义 secretKey，一般用于后端自行加解密，如果是前端加密后端解密，则需要自定义secretKey，secretKey一致才能正确解密
System.out.println("经过ECB加密的密文为：" + SM4Utils.encryptData_ECB("123456"));
System.out.println("经过ECB解密的密文为：" + SM4Utils.decryptData_ECB("UQZqWWcVSu7MIrMzWRD/wA=="));
//使用自定义 secretKey，传入的 secretKey 必须为16位，可包含字母、数字、标点
System.out.println("经过ECB加密的密文为：" + SM4Utils.encryptData_ECB("123456"));
System.out.println("经过ECB解密的密文为：" + SM4Utils.decryptData_ECB("UQZqWWcVSu7MIrMzWRD/wA=="));

// CBC 加密模式（更加安全）需要两个密钥 secretKey 和 iv
System.out.println("经过CBC加密的密文为：" + SM4Utils.encryptData_CBC("123456"));
System.out.println("经过CBC解密的密文为：" + SM4Utils.decryptData_CBC("hbMK6/IeJ3UTzaTgLb3f3A=="));
//同样可以自定义 secretKey 和 iv，需要两个密钥前后端都一致
System.out.println("经过CBC加密的密文为：" + SM4Utils.encryptData_CBC("123456", "asdfghjklzxcvb!_", "1234567890123456"));
System.out.println("经过CBC解密的密文为：" + SM4Utils.decryptData_CBC("sTyCl3G6TF311kIENzsKNg==", "asdfghjklzxcvb!_", "1234567890123456"));
```

[SM4前后端加解密下载链接](https://download.csdn.net/download/qq_48922459/85001193)
有很多用户反映CSDN这个资源要什么下载码，我下面放一个github的仓库，里面有文件和演示demo
[SM4前后端加解密Demo Github地址](https://github.com/XuYijie000416/SM4EncryptAndDecrypt)

# 一、前端加密输入的密码
##  前置检测

我使用 Vue 项目做示范吧，一个很==重要==的事情，如果的项目有 `ESLint` ，或者你使用的是 Vue3 或者 Vue-cli3 以上的项目，ESLint 应该都会默认开启，如果没有这个更好。

看一看你的项目根目录，应该会有一个 `.eslintrc.js` 文件，添加 rules 和 'globals'，即使我已经很努力的修改代码了，但是还是有一个方法 “base64js” 会报错，所以我们把这个方法忽略掉。

![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/SM4加密0.png)

```javascript
rules: {
    'no-console': process.env.NODE_ENV === 'production' ? 'warn' : 'off',
    'no-debugger': process.env.NODE_ENV === 'production' ? 'warn' : 'off',
  },
  "globals": {
    "base64js": true,
  }
```



##  粘贴我的 SM4.js 代码
> 前端的 SM4 加解密我已经上传到 npm 仓库了，你们可以`npm install sm4util`直接安装使用，无需进行下载代码
```js
//引入和使用
import {SM4Util} from "sm4util";
const sm4 = new SM4Util();
```


<font color=#999AAA >这个 `sm4.js`就是加密用的 ，位置随意放，代码太长了，我上传到文件了，前后端的文件给你们放一起了，不需要积分。

[SM4前后端加解密下载链接](https://download.csdn.net/download/qq_48922459/85001193)
有很多用户反映CSDN这个资源要什么下载码，我下面放一个github的仓库，里面有文件和演示demo
[Github地址](https://github.com/XuYijie000416/SM4EncryptAndDecrypt)




![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/SM4加密1.png)




##  在组件中调用

<font color=#999AAA >如图，引入 `import {SM4Util} from '@/utils/sm4';` from后面的路径自己调整
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/SM4加密2.png)

<font color=#999AAA >如图，开始加密输入框的密码，并传输到后端，mounted 这样写可以在控制台直接输出 123456 的加密结果。传输到后端我就不演示了。

```html
<template>
	<div>
		<input v-model="mobilePhone" placeholder="请输入手机号">
		<input v-model="password" placeholder="请输入密码">
	</div>
</template>
<script>
import {SM4Util} from '@/utils/sm4';
export default {
  name: "Register",
  data(){
    return{
      username: '',
      mobilePhone: '',
      password: '',
      rePassword: ''
    }
  },
  mounted() {
    const sm4 = new SM4Util();
    const test = sm4.encryptData_CBC('123456')
    console.log('123456的加密结果：' + test)
  },
  
methods:{
    register(){
        // sm4加密
        const sm4 = new SM4Util();
        this.$axios
            .post("/user/register", {
              mobilePhone: this.mobilePhone.trim(),
              password: sm4.encryptData_CBC(this.password.trim()),
            })
      }
    }
}
</script>
```

##  加密结果
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/SM4加密3.png)


---


# 二、Springboot 后端进行 SM4 的解密和加密

> 后端的 SM4 加解密我已经上传到 maven 中央仓库了，你们可以直接引入到 pom，无需进行下面的 commons-codec 引入和复制代码操作
```xml
<dependency>
    <groupId>icu.xuyijie</groupId>
    <artifactId>SM4Utils</artifactId>
    <version>1.4.8</version>
</dependency>
```

---

> 下面是一些源码示例

## 引入库


<font color=#999AAA >需要引入一个依赖

```xml
<dependency>
  <groupId>commons-codec</groupId>
    <artifactId>commons-codec</artifactId>
    <version>1.16.0</version>
</dependency>
```

##  复制我的 SM4 加解密代码

<font color=#999AAA >一共有 4 个文件，我直接给你们下载吧，不需要积分，下载后在项目里放在一起
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/SM4加密4.png)

[SM4前后端加解密下载链接](https://download.csdn.net/download/qq_48922459/85001193)
有很多用户反映CSDN这个资源要什么下载码，我下面放一个github的仓库，里面有文件和演示demo
[SM4前后端加解密Demo Github地址](https://github.com/XuYijie000416/SM4EncryptAndDecrypt)


---


## 调用方法

<font color=#999AAA >在 `SM4Utils` 那个文件里面有一个 main 方法，我写好了，运行

![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/SM4加密5.png)


<font color=#999AAA >看，和，和前端的加密结果一样，也可以解密出来，秘诀就是前面说的前后端的“钥匙要一样”
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/SM4加密6.png)

<font color=#999AAA >ECB 和 CBC 的区别你们可以自己了解一些，反正用 CBC 就完事了，据说更安全

---

# 总结
<font color=#999AAA >这是一个总结
