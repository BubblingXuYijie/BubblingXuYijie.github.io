---
title: Python 调用高德 API 实现地址转为经纬度
date: 2021-12-31 10:02:58
categories: Python
tags:
    - Python
    - 高德API
cover: https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/PythonLogo.jpg
---
# 前言

<font color=#999AAA >偶然接触到这个东西，记录一下</font>

<hr style=" border:solid; width:100px; height:1px;" color=#000000 size=1">


# 一、申请高德的Key

[高德开放平台网址](https://lbs.amap.com/)


<font color=#999AAA >注册登录后，点击右上角头像旁边的控制台

<font color=#999AAA >点击“我的应用”，然后右边有个“创建新应用”，点击后输入一些内容，就创建成功了，`key` 就获取到了

![在这里插入图片描述](https://img-blog.csdnimg.cn/788630f95a354b38bb93f272ec2979fd.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA57mB5Y2O5bC95aS05ruh5piv5q6H,size_20,color_FFFFFF,t_70,g_se,x_16)

<hr style=" border:solid; width:100px; height:1px;" color=#000000 size=1">





# 二、要转换的地址放入文件

<font color=#999AAA >可选 csv、xlsx、txt，推荐 txt ，因为 txt 无论如何都不会报错，csv 和 xlsx 高概率会报 UTF-8 编码无法转换的错，网上的解决方案都是修改编码，这样不行，所以还是 txt 吧

一个地址换一行，如果用 xlsx 的话也一样，一个地址换一行


![在这里插入图片描述](https://img-blog.csdnimg.cn/8b8e404a324649e48279b28a43deb703.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA57mB5Y2O5bC95aS05ruh5piv5q6H,size_8,color_FFFFFF,t_70,g_se,x_16)

<hr style=" border:solid; width:100px; height:1px;" color=#000000 size=1">


# 三、编写代码

<font color=#999AAA >这代码也不是我本人写的，我也不怎么会 Python ，所以没有注释讲解。。。

<font color=#999AAA >将下面的 `key` 替换成你刚刚创建的新应用下面的 key，然后地址文件改成你们刚刚创建的 `txt`




```python
import requests
import codecs
from openpyxl import Workbook

wb = Workbook()
sheet = wb.active


def get_location(address, i):
    print(i)
    # 这是高德的API接口地址
    url = "https://restapi.amap.com/v3/geocode/geo"
    data = {
        # 替换为你在高德地图开发者平台申请的key
        'key': 'a8c3f94ebc2fea82c283effa241aa066',
        'address': address
    }
    r = requests.post(url, data=data).json()
    sheet["A{0}".format(i)].value = address.strip('\n')
    print(r)
    if r['status'] == '1':
        if len(r['geocodes']) > 0:
            GPS = r['geocodes'][0]['location']
            sheet["B{0}".format(i)].value = '[' + GPS + ']'
        else:
            sheet["B{0}".format(i)].value = '[]'
    else:
        sheet["B{0}".format(i)].value = '未找到'


# 将地址信息替换为自己的文件，一行代表一个地址，根据需要也可以自定义分隔符
# 读取地址文件。我的为D:\下载\Documents\地点.txt
f = codecs.open(r"D:\下载\Documents\地点.txt", "r", "utf-8")
i = 0
while True:
    line = f.readline()
    i = i + 1
    if not line:
        f.close()
        # 设置保存路径
        wb.save(r"D:\下载\Documents\经纬度信息.xlsx")
        break
    get_location(line, i)

```

<hr style=" border:solid; width:100px; height:1px;" color=#000000 size=1">

# 四、运行结果

<font color=#999AAA >你设置的路径下生成了结果文件，打开看看

![在这里插入图片描述](https://img-blog.csdnimg.cn/3535b3b23e7340fdbb5819183972f611.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/a41e5df7d9a64fdea96c704f5c11d1f9.png)

<font color=#999AAA >准确度还挺高，这种直接写小区名和学校名都能识别到，高德 niubility



<hr style=" border:solid; width:100px; height:1px;" color=#000000 size=1">

# 总结
