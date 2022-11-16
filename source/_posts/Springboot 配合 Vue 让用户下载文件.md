---
title: Springboot 配合 Vue 让用户下载文件
date: 2022-02-25 08:49:41
categories: Java
tags:
    - SpringBoot
    - Vue
cover: /img/springbootLogo.jpeg
---
# 前言

<font color=#999AAA >创建一个 Springboot 项目，也可以是普通 Java 项目，前端用 Vue 的 axios 接收下载</font>

<hr style=" border:solid; width:100px; height:1px;" color=#000000 size=1">

# 一、后端


<font color=#999AAA >解析都在注释里

```java
@RequestMapping("/download")
    public void download(HttpServletResponse response) throws Exception {
   		//这里写要让前端下载的文件的路径
        File file = new File("F:/uploadFiles/test.xlsx");
        //设置编码格式，防止下载的文件内乱码
        response.setCharacterEncoding("UTF-8");
        //获取路径文件对象
        String realFileName = file.getName();
        //设置响应头类型，这里可以根据文件类型设置，text/plain、application/vnd.ms-excel等
        response.setHeader("content-type", "application/octet-stream;charset=UTF-8");
        response.setContentType("application/octet-stream;charset=UTF-8");
        //如果不设置响应头大小，可能报错：“Excel 已完成文件级验证和修复。此工作簿的某些部分可能已被修复或丢弃”
        response.addHeader("Content-Length", String.valueOf(file.length()));
        try 
        	//Content-Disposition的作用：告知浏览器以何种方式显示响应返回的文件，用浏览器打开还是以附件的形式下载到本地保存
            //attachment表示以附件方式下载   inline表示在线打开   "Content-Disposition: inline; filename=文件名.mp3"
            // filename表示文件的默认名称，因为网络传输只支持URL编码的相关支付，因此需要将文件名URL编码后进行传输,前端收到后需要反编码才能获取到真正的名称
            response.setHeader("Content-Disposition", "attachment;filename=" + java.net.URLEncoder.encode(realFileName.trim(), "UTF-8"));
        } catch (UnsupportedEncodingException e1) {
            e1.printStackTrace();
        }
        //初始化文件流字节缓存
        byte[] buff = new byte[1024];
        try {
       		//开始写入
            OutputStream os = response.getOutputStream();
            //写入完成，创建文件流
            BufferedInputStream bis = new BufferedInputStream(new FileInputStream(file));
            // bis.read(data)：将字符读入数组。在某个输入可用、发生I/O错误或者已到达流的末尾前，此方法一直阻塞。
            // 读取的字符数，如果已到达流的末尾，则返回 -1
            int i = bis.read(buff);
            while (i != -1) {
                os.write(buff, 0, buff.length);
                os.flush();
                i = bis.read(buff);
            }
        }catch (Exception e){
            e.printStackTrace();
        }finally {
            if (bis != null) {
                try {
                    bis.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
```





# 二、前端接收下载


<font color=#999AAA >我这里使用的是 axios，你们如果不是需要修改一下，但是下载文件的方法代码都是一样的

```javascript
download(){
	const config = {
        method: 'get',
        url: 'http://localhost:19021/api/fwq/fwqProduct/downloadFile',
        headers: {
          //和后端设置的一样
          'Content-Type': 'application/octet-stream;charset=UTF-8'
        },
        responseType: 'blob'
      };
      axios(config).then(response => {
        const url = window.URL.createObjectURL(new Blob([response.data]));
        const link = document.createElement('a');
        link.href = url;
        link.setAttribute('download', 'xxxx.pdf');
        document.body.appendChild(link);
        link.click();
      })
},


```


<hr style=" border:solid; width:100px; height:1px;" color=#000000 size=1">

