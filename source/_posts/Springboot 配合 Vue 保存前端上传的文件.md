---
title: Springboot 配合 Vue 保存前端上传的文件
date: 2022-02-25 10:42:49
categories: Java
tags:
- SpringBoot
- Vue
cover: https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/springbootLogo.jpeg
---
# 前言

<font color=#999AAA >新建一个 Springboot 项目，前端我使用原生 html 来演示上传，你们用 element 或者其他什么的比较方便。

<hr style=" border:solid; width:100px; height:1px;" color=#000000 size=1">

# 一、后端代码
```java
@RequestMapping("/upload")
public void upload(MultipartFile multipartFile) throws IOException {
    //获取前端上传的文件
    String fileName = multipartFile.getOriginalFilename();
    //创建文件对象，用于保存文件到该路径
    File file = new File("C:\\Users\\DELL\\Desktop\\文件\\" + fileName);
    //如果上面的路径文件夹没有，则创建文件夹
    if(!file.exists()) {
       //先得到文件的上级目录，并创建上级目录，在创建文件
       file.getParentFile().mkdir();
        try {
            //创建文件
            file.createNewFile();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    //将上传的文件保存到路径下
    multipartFile.transferTo(file);
}
```




# 二、前端

<font color=#999AAA >自行安装axios，也可以不用axios

```javascript
<template>
	<div
		<input type="file" ref="file" @change="upload()">		
	</div>
</template>
<script>
	export default{
		methods:{
			upload(){
				var that = this;
				// 获取到this 赋值个that
				var file = this.$refs.file.files[0];
				// 如果没有文件就返回
			    if(!file){return}
				var data = new FormData();
				// 将文件加入到data中
				data.append("file",file);
				axios.post(
					"/upload",
					data,
				)
				.then(res=>{
					consolo.log(res.data)
				}	
				// 清空文件列表
				this.$refs.file.value=""；
				})
			},
		},
	}

```



<hr style=" border:solid; width:100px; height:1px;" color=#000000 size=1">

