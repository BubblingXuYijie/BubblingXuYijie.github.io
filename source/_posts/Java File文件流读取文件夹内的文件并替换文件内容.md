---
title: Java File文件流读取文件夹内的文件并替换文件内容
date: 2022-02-24 17:16:15
categories: Java
tags:
    - Java
cover: https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/javaLogo.png
---
# 一、前言
<font color=#999AAA >批量读取文件夹内的文件，并替换各个文件的内容


# 二、代码


<font color=#999AAA >新建一个普通 Java 项目就可以，创建文件 ReadFile.java

```java
import java.io.*;

public class ReadFile {
    public void replaceFileStr() {
        //遍历文件夹内所有内容，不包换文件夹里的文件夹里的内容
        String path = "F:\\工作和作业\\Java\\IDEA项目\\读取文件替换文件内容\\txt";
        //获取其file对象
        File file = new File(path);
        //遍历path下的文件和目录，放在File数组中
        File[] fileArray = file.listFiles();
        //判断文件夹内是否有文件
        if (fileArray != null){
            //遍历File[]数组
            for(File f : fileArray){
                String filepath = f.getPath();
                //要替换的旧内容
                String oldStr = "1010";
                //替换成的新内容
                String newtStr = "哈哈哈";
                try {
                    // 创建文件输入流
                    FileReader fileReader = new FileReader(filepath);
                    // 创建缓冲字符数组
                    char[] data = new char[1024];
                    StringBuilder sb = new StringBuilder();
                    // fis.read(data)：将字符读入数组。在某个输入可用、发生I/O错误或者已到达流的末尾前，此方法一直阻塞。
                    // 读取的字符数，如果已到达流的末尾，则返回 -1
                    int rn = 0;
                    while ((rn = fileReader.read(data)) > 0) { // 读取文件内容到字符串构建器
                        String str = String.valueOf(data, 0, rn);// 把数组转换成字符串
                        sb.append(str);
                    }
                    // 生成字符串，并替换搜索文本
                    String str = sb.toString().replace(oldStr, newtStr);
                    // 创建文件输出流
                    FileWriter fileWriter = new FileWriter(filepath);
                    // 把替换完成的字符串写入文件内
                    fileWriter.write(str.toCharArray());
                    // 关闭文件流，释放资源
                    fileReader.close();
                    fileWriter.close();
                    //若非目录(即文件)，则打印，提示替换完成
                    if(!f.isDirectory()) {
                        System.out.println(f.getPath() + "已完成替换。");
                    }
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }else {
            System.out.println("文件夹为空！");
        }
    }

    public static void main(String[] args) {
        ReadFile readFile = new ReadFile();
        readFile.replaceFileStr();
    }
}

```

<hr style=" border:solid; width:100px; height:1px;" color=#000000 size=1">



# 三、运行结果

<font color=#999AAA >我新建了一个 test.txt ，里面写了 `1010嘻嘻嘻`，你们可以多建几个，可以批量读取替换的
![在这里插入图片描述](https://img-blog.csdnimg.cn/f514471595fb41febaacdd3dad213792.png)
<font color=#999AAA >运行项目，发现 `1010` 被替换为了 `哈哈哈`

![在这里插入图片描述](https://img-blog.csdnimg.cn/988f5be721cb4a3ca21ddb4881ee5f70.png)




<hr style=" border:solid; width:100px; height:1px;" color=#000000 size=1">

