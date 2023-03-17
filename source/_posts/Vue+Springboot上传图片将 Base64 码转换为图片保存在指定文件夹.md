---
title: Vue+Springboot上传图片将 Base64 码转换为图片保存在指定文件夹
date: 2021-12-21 10:37:43
categories: Java
tags: 
    - Vue
    - SpringBoot
    - Base64
cover: https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/Base640.png
---
# 前言

<font color=#999AAA >整体思路：

vant插件默认的图片从前端上传到后端，是以 base64 码的形式传输的，而 base64 码很长很长，有几千个字符，存入数据库会占用空间，每次读取都传输 base64 的话也很消耗带宽和流量，所以后端接收以后要转化 base64 为图片存到硬盘或服务器，而数据库只保存图片的物理地址的路径，例如 F:/img/aa.png</font>

<hr style=" border:solid; width:100px; height:1px;" color=#000000 size=1">


# 一、Vue 前端

<font color=#999AAA >我用的时 vant 的上传框架，你们用js什么的把图片转化为 base64 码。

```html
<van-uploader :after-read="uploadImg" preview-size="50px" v-model="userHead" :max-count="1" />

<script>
methods:{
    submit() {
        this.$axios
            .post("/user/userInfo", {
                //userHead传给后台的是一串base64，后台要进行转化
                userHead: this.file,
                username: this.username,
            })
            .then(successResponse => {
                console.log(successResponse)
            })
    }
}
</script>

```

<hr style=" border:solid; width:100px; height:1px;" color=#000000 size=1">


# 一.五 注意！！！
==注意！！！==
> 1、现在我已经将把 `base64` 转换成文件保存到指定位置的代码上传到了 `maven 中央仓库`，你们可以直接引入这个依赖使用
> 2、如果想学习源码，那么就继续往下面看二，二里面是我最简化的代码，实际我封装的方法要更复杂，支持更多传参类型和容错，比如可以自动判断文件类型而你不需要传入文件后缀名，接受包含或以 '/' 、'\\' 结尾或者以 '' 结尾的路径格式，增加方法，将文件转换为 base64 码，你可以传入文件路径或者二进制文件流。
> 3、下面是引入依赖和使用方法

```xml
<!-- https://mvnrepository.com/artifact/icu.xuyijie/Base64Utils -->
<dependency>
    <groupId>icu.xuyijie</groupId>
    <artifactId>Base64Utils</artifactId>
    <version>1.2.1</version>
</dependency>
```
```java
// 将文件编码成Base64，可传入文件全路径，或者一个 File 对象
String s = Base64Util.transferToBase64("D:/下载/Screenshot_20221008-090627.png");
File file = new File(filePath);
String s = Base64Util.transferToBase64(file);
System.out.println(s);
// 将Base64转换成文件保存到指定位置，可传入文件全路径或者分别传入保存位置和文件名
String s1 = Base64Util.generateFile(s, "D:/下载/aaa.png");
String s1 = Base64Util.generateFile(s, "D:/下载", "aaa.png");
System.out.println(s1);
```


# 二、Springboot 后端



##  引入Base64依赖

```xml
<!--        工具（包含Base64）-->
        <dependency>
            <groupId>commons-codec</groupId>
            <artifactId>commons-codec</artifactId>
            <version>1.10</version>
        </dependency>
```



##  在项目里新建 utils 包，新建一个Base64Util类

```java
package pers.xuyijie.communityinteractionsystem.utils;

import org.apache.commons.codec.binary.Base64;
import java.io.*;

public class Base64Util {
    // 对字节数组字符串进行Base64解码并生成图片
    //imgFilePath 待保存的本地路径
    public static void GenerateImage(String base64Str, String imgFilePath) {
        if (base64Str == null) // 图像数据为空
            return;
        try {
            // Base64解码
            byte[] bytes = Base64.decodeBase64(base64Str);
            for (int i = 0; i < bytes.length; ++i) {
                if (bytes[i] < 0) {// 调整异常数据
                    bytes[i] += 256;
                }
            }
            // 生成jpeg图片
            OutputStream out = new FileOutputStream(imgFilePath);
            out.write(bytes);
            out.flush();
            out.close();
            //====
        } catch (Exception e) {
        }
    }
}

```



<font color=#999AAA >Service 代码如下：

==String base64 = StringUtils.substringAfter(userInfo.getUserHead(),";base64,");== 这句是把 base64 码的前缀去掉


==path== 里面就写你要保存图片的位置，和文件名，意思是保存到 F 盘根目录，文件名是以用户名+当前时间毫秒数命名的。

==Base64Util.GenerateImage(base64,path);== 是用我们的工具把前端传来的 base64 解析成图片保存在path定义的路径下



==File file = new File(path)；userInfo1.setUserHead(file.getPath());== 就是保存 path 路径到数据库


```java
@Override
    public ResultCode userInfo(UserInfo userInfo, HttpServletRequest request) {
        HttpSession session = request.getSession();
        User user = (User) session.getAttribute("user");
        String mobilePhone = user.getMobilePhone();
        QueryWrapper<UserInfo> selectQuery = new QueryWrapper();
        selectQuery.eq("mobile_phone", mobilePhone);
        //用户表和用户信息表里面的mobile_phone是一一对应的
        UserInfo userInfo1 = userInfoMapper.selectOne(selectQuery);
        //处理前端传来的base64为文件路径
        String base64 = StringUtils.substringAfter(userInfo.getUserHead(),";base64,");
        //保存到设置的路径下，文件名手机号加毫秒命名
        String path = "F:\\" + user.getMobilePhone() + System.currentTimeMillis() + ".png";
        Base64Util.GenerateImage(base64,path);
        File file = new File(path);
        userInfo1.setUserHead(file.getPath());
        userInfo1.setUsername(userInfo.getUsername());
        userInfo1.setAge(userInfo.getAge());
        userInfo1.setSex(userInfo.getSex());
        userInfo1.setHomeTown(userInfo.getHomeTown());
        userInfo1.setEditTime(new Date());
        userInfoMapper.updateById(userInfo1);
        user.setUsername(userInfo.getUsername());
        userMapper.updateById(user);
        return new ResultCode(200);
    }
```

![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/Base640.png)
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/Base641.png)

## 2.读取的话就不写了，看我的另一篇，另一篇怎么从服务器读取也有教

传送门：[Springboot物理地址映射+Nginx静态资源代理实现前端上传并访问服务器图片](https://blog.csdn.net/qq_48922459/article/details/122035967?spm=1001.2014.3001.5501)





<hr style=" border:solid; width:100px; height:1px;" color=#000000 size=1">

# 总结

