---
title: Springboot 引入 EasyExcel 写入数据到流 直接让前端下载（Vue + axios请求）
date: 2022-01-17 08:49:12
categories: Java
tags:
    - SpringBoot
    - EasyExcel
    - Vue
    - axios
cover: /img/EasyexcelLogo.jpg
---
#  前言
easyexcel重写了poi对07版Excel的解析，能够原本一个3M的excel用POI sax依然需要100M左右内存降低到几M，并且再大的excel不会出现内存溢出，03版依赖POI的sax模式。在上层做了模型转换的封装，让使用者更加简单方便。

有时我们的用户需要导出他们查询到的数据，这时，就可以使用easyexcel直接将数据写入流，提供下载。


# 一、Springboot 引入 EasyExcel 依赖
<font color=#999AAA >如果你在网上看到还要引入 poi 什么的依赖，那是针对旧版 easyexcel ，现在的版本不需要引入其他依赖，就下面的就可以了。

```xml
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>easyexcel</artifactId>
    <version>3.0.5</version>
</dependency>
```


<hr style=" border:solid; width:100px; height:1px;" color=#000000 size=1">

# 二、后端代码

```java
@RequestMapping("/download")
public void download(HttpServletResponse response) throws IOException {
    // 这里的 ContentType 要和前端请求携带的 ContentType 相对应
    response.setContentType("application/vnd.ms-excel");
    response.setCharacterEncoding("utf-8");
    // 这里URLEncoder.encode可以防止中文乱码 当然和easyexcel没有关系
    String fileName = URLEncoder.encode("测试", "UTF-8");
    response.setHeader("Content-disposition", "attachment;filename=" + fileName + ".xlsx");
    // DownloadData 是实体类，sheet 里面是 sheet 名称，doWrite 里面放要写入的数据，类型为 List<DownloadData>
    EasyExcel.write(response.getOutputStream(), DownloadData.class).autoCloseStream(Boolean.FALSE).sheet("模板").doWrite(downloadDataList);
}
```

<hr style=" border:solid; width:100px; height:1px;" color=#000000 size=1">

# 三、前端代码

```javascript
getExcel(){
		//你们的函数不一定是 axios ，根据你们自己配置的来
		axios({
			headers: {
				//这里和后端的相对应
				"Content-Type": "application/vnd.ms-excel",
			},
			//这里也可以是 blob
			responseType: "arraybuffer"
			method: "post",
			url: '/download',
			// 请求参数，可以为空
			params: {
				
			},
		}).then(successResponse=> {
		//这里面是请求成功以后下载文件的方法
			let objectUrl = successResponse
			const blob = new Blob([successResponse], {'application/vnd.ms-excel'});
			objectUrl = URL.createObjectURL(blob);
			const a = document.createElement("a");
			a.href = objectUrl;
			//这里可以重新定义文件名，以前端为准
			a.download = "test.xlsx";
			a.click();
			a.remove();
			URL.revokeObjectURL(objectUrl)
		});
	},

```





<hr style=" border:solid; width:100px; height:1px;" color=#000000 size=1">

#  四、PostMan测试方法
![在这里插入图片描述](https://img-blog.csdnimg.cn/6335f699e08e4e45b3b2b8d5d5cab061.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA57mB5Y2O5bC95aS05ruh5piv5q6H,size_20,color_FFFFFF,t_70,g_se,x_16)
![在这里插入图片描述](https://img-blog.csdnimg.cn/347976639de341c0ac7ac6ee3397a9de.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA57mB5Y2O5bC95aS05ruh5piv5q6H,size_20,color_FFFFFF,t_70,g_se,x_16)
![在这里插入图片描述](https://img-blog.csdnimg.cn/1dc19202dfbe4397948acd479fb8562e.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA57mB5Y2O5bC95aS05ruh5piv5q6H,size_20,color_FFFFFF,t_70,g_se,x_16)

