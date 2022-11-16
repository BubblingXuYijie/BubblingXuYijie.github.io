---
title: EasyExcel 样式注解大全
date: 2022-04-08 08:55:04
categories: Java
tags:
    - SpringBoot
    - EasyExcel
cover: /img/EasyexcelLogo.jpg
---
# 前言
>太详细了，转载做笔记用，原文作者如下，我又加了点自己的详细示例

https://www.cnblogs.com/bluekang/p/13438666.html#!comments
https://www.cnblogs.com/Brainpan/p/5804167.html
![在这里插入图片描述](https://img-blog.csdnimg.cn/43a01f80e22a4dfdbfcdd6dade8f7074.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/82038780caea4d088e2686c729307f5e.png)

---



# 11个注解
>@ExcelProperty
@ColumnWith 列宽
@ContentFontStyle 文本字体样式
@ContentLoopMerge 文本合并
@ContentRowHeight 文本行高度
@ContentStyle 文本样式
@HeadFontStyle 标题字体样式
@HeadRowHeight 标题高度
@HeadStyle 标题样式
@ExcelIgnore 忽略项
@ExcelIgnoreUnannotated 忽略未注解

# 字段注解	类注解
`@ColumnWith(列宽)`
`@ColumnWidth(全局列宽)`
`@ExcelProperty(字段配置)`
`@HeadFontStyle(头样式)`
`@HeadRowHeight(标题高度)`
`@ContentFontStyle(内容字体样式)`
`@ContentRowHeight(内容高度)`
`@ExcelProperty`
`必要的一个注解，注解中有三个参数value,index,converter分别代表列明，列序号，数据转换方式
value和index只能二选一，通常不用设置converter`
`1.value 通过标题文本对应`
`2.index 通过文本行号对应`
`3.converter 转换器，通常入库和出库转换使用，如性别入库0和1，出库男和女`



```java
public class ImeiEncrypt {
    @ExcelProperty(value = "值")
    private String valueField;

    @ExcelProperty(value = 1,converter =IndustryIdConverter.class)
    private String indexField;

    @ExcelProperty(value = "值对应和转换器",converter =IndustryIdConverter.class)
    private String valueAndConverterField;
}
```


`@ColumnWith
设置列宽度,只有一个参数value，value的单位是字符长度，最大可以设置255个字符，因为一个excel单元格最大可以写入的字符个数就是255个字符。
`

```java
public class ImeiEncrypt {
    @ColumnWidth(value = 18)
    private String imei;
}
```

`@ContentFontStyle
用于设置单元格内容字体格式的注解`

>参数：
fontName	字体名称
fontHeightInPoints	字体高度
italic	是否斜体
strikeout	是否设置删除水平线
color	字体颜色
typeOffset	偏移量
underline	下划线
bold	是否加粗
charset	编码格式

`@ContentLoopMerge
用于设置合并单元格的注解`

>参数：
eachRow
columnExtend

`@ContentRowHeight
用于设置行高`

>参数：
value	行高，-1代表自动行高

`@ContentStyle
设置内容格式注解`

>参数：
dataFormat	日期格式
hidden	设置单元格使用此样式隐藏
locked	设置单元格使用此样式锁定
quotePrefix	在单元格前面增加`符号，数字或公式将以字符串形式展示
horizontalAlignment	设置是否水平居中
wrapped	设置文本是否应换行。将此标志设置为true通过在多行上显示使单元格中的所有内容可见
verticalAlignment	设置是否垂直居中
rotation	设置单元格中文本旋转角度。03版本的Excel旋转角度区间为-90°90°，07版本的Excel旋转角度区间为0°180°
indent	设置单元格中缩进文本的空格数
borderLeft	设置左边框的样式
borderRight	设置右边框样式
borderTop	设置上边框样式
borderBottom	设置下边框样式
leftBorderColor	设置左边框颜色
rightBorderColor	设置右边框颜色
topBorderColor	设置上边框颜色
bottomBorderColor	设置下边框颜色
fillPatternType	设置填充类型
fillBackgroundColor	设置背景色
fillForegroundColor	设置前景色
shrinkToFit	设置自动单元格自动大小

`@HeadFontStyle
用于定制标题字体格式`

>参数	含义
fontName	设置字体名称
fontHeightInPoints	设置字体高度
italic	设置字体是否斜体
strikeout	是否设置删除线
color	设置字体颜色
typeOffset	设置偏移量
underline	设置下划线
charset	设置字体编码
bold	设置字体是否加粗

`@HeadRowHeight
设置标题行行高`

>参数	含义
value	设置行高，-1代表自动行高

`@HeadStyle
设置标题样式`

>参数	含义
dataFormat	日期格式
hidden	设置单元格使用此样式隐藏
locked	设置单元格使用此样式锁定
quotePrefix	在单元格前面增加`符号，数字或公式将以字符串形式展示
horizontalAlignment	设置是否水平居中
wrapped	设置文本是否应换行。将此标志设置为true通过在多行上显示使单元格中的所有内容可见
verticalAlignment	设置是否垂直居中
rotation	设置单元格中文本旋转角度。03版本的Excel旋转角度区间为-90°90°，07版本的Excel旋转角度区间为0°180°
indent	设置单元格中缩进文本的空格数
borderLeft	设置左边框的样式
borderRight	设置右边框样式
borderTop	设置上边框样式
borderBottom	设置下边框样式
leftBorderColor	设置左边框颜色
rightBorderColor	设置右边框颜色
topBorderColor	设置上边框颜色
bottomBorderColor	设置下边框颜色
fillPatternType	设置填充类型
fillBackgroundColor	设置背景色
fillForegroundColor	设置前景色
shrinkToFit	设置自动单元格自动大小


`@ExcelIgnore
不将该字段转换成Excel`

`@ExcelIgnoreUnannotated
没有注解的字段都不转换`

#  基础综合示例

```java
//行高全部设为40
@HeadRowHeight(value = 40)
//标题全部居中
@HeadStyle(horizontalAlignment = CENTER)
public class SupervisionDailyExportProcessDTO {
	//"二、问题整改情况"是大标题，"序号"是大标题下面的子标题
    @ExcelProperty({"二、问题整改情况", "序号"})
    @ContentStyle(horizontalAlignment = CENTER)
    private Integer id;
    @ExcelProperty({"二、问题整改情况", "问题来源"})
    //单独设置这一列列宽为40
    @ColumnWidth(value = 40)
    @ContentStyle(wrapped = true, horizontalAlignment = CENTER)
    private String questionOrigin;
    @ExcelProperty({"二、问题整改情况", "督察点位"})
    @ColumnWidth(value = 40)
    @ContentStyle(wrapped = true, horizontalAlignment = CENTER)
    private String supervisionPoint;
    @ExcelProperty({"二、问题整改情况", "问题内容"})
    @ColumnWidth(value = 40)
    @ContentStyle(wrapped = true, horizontalAlignment = CENTER)
    private String questionContents;
    @ExcelProperty({"二、问题整改情况", "责任单位"})
    @ColumnWidth(value = 30)
    @ContentStyle(horizontalAlignment = CENTER)
    private String orgName;
    @ExcelProperty({"二、问题整改情况", "整改情况"})
    @ColumnWidth(value = 40)
    @ContentStyle(wrapped = true, horizontalAlignment = CENTER)
    private String processSituation;
}
```


# 补充颜色
>NPOI Excel 单元格颜色对照表，在引用了 NPOI.dll 后可通过 ICellStyle 接口的 FillForegroundColor 属性实现 Excel 单元格的背景色设置，FillPattern 为单元格背景色的填充样式。
NPOI Excel 单元格背景颜色设置方法以及颜色对照表：

```java
ICellStyle style = workbook.CreateCellStyle();
style.FillForegroundColor = NPOI.HSSF.Util.HSSFColor.Red.Index;
style.FillPattern = FillPattern.SolidForeground;
 
ICell cell = workbook.CreateSheet().CreateRow(0).CreateCell(0);
cell.CellStyle = style;
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/2d743a2dbdd04b12a1113d12d90bfc89.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA57mB5Y2O5bC95aS05ruh5piv5q6H,size_12,color_FFFFFF,t_70,g_se,x_16)
![在这里插入图片描述](https://img-blog.csdnimg.cn/b5b12a894b524dd0829487194c325c39.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA57mB5Y2O5bC95aS05ruh5piv5q6H,size_12,color_FFFFFF,t_70,g_se,x_16)


---

# 总结

