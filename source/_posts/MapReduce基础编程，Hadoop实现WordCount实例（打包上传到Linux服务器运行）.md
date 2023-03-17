---
title: MapReduce基础编程，Hadoop实现WordCount实例（打包上传到Linux服务器运行）
date: 2021-04-29 18:57:10
categories: Linux
tags:
    - MapReduce
    - Linux
    - Hadoop
    - WordCount
cover: https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/MapReduce0.png
---
#  没有配置好环境的同学看我的传送门

[Deepin（Ubuntu通用）安装Hadoop伪分布环境（集成Hbase、Hive、MySQL、Spark、Scala）](https://blog.csdn.net/qq_48922459/article/details/115515516)

<hr>


# 1、在Windows下下载Hadoop
[Hadoop下载地址](https://www.apache.org/dyn/closer.cgi/hadoop/common/hadoop-3.2.2/hadoop-3.2.2.tar.gz)

bfsu这个镜像最快，清华的经常断
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/MapReduce1.png)

下载完成后解压
<hr>

# 2、使用idea新建Java工程
新建一个普通的java项目就行，jdk最好用1.8，不然有可能会报错

首先在新建的项目文件夹里新建一个文件夹，存放要导入的jar包
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/MapReduce2.png)
将下载的Hadoop解压后share里面的这五个文件夹里的jar包全部粘贴到项目目录下新建的“引入的jar包”内
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/MapReduce3.png)
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/MapReduce4.png)

在 idea 中配置引入的 jar 包

![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/MapReduce5.png)

选择自己新建的存放 jar 包的文件夹

![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/MapReduce6.png)
<hr>

# 3、编写代码
新建 WordCount java 文件
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/MapReduce7.png)

代码如下



```java
import java.io.IOException;

import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.fs.FileSystem;

import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;


public class WordCount_final
{
    public WordCount_final() {}
    //第一个表示输入key的类型；第二个Text表示输入value的类型；第三个Text表示表示输出键的类型；第四个IntWritable表示输出值的类型
    public static class WordCountMapper extends Mapper<LongWritable, Text, Text, IntWritable>
    {
        //对数据进行打散
        public void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException
        {
//			System.out.println("Map function: " + value);
            //1.接收到的每行数据转换成字符串
            String line = value.toString();
            //2.按照指定分隔符进行分隔
            String[] words = line.split(" ");	// 注意此处的分割符
            //3.输出结果以<key,value>形式
            for(String w : words)
            {
                //写到Reducer端
                context.write(new Text(w), new IntWritable(1));
            }
        }
    }
    //自定义一个reduce类，归并
    public static class WordCountReducer extends Reducer<Text, IntWritable, Text, IntWritable>
    {
        //key是单词，values是次数
        public void reduce(Text key, Iterable<IntWritable> values, Context context) throws IOException, InterruptedException
        {
            //1.reduce参数中values是一个集合，表示相同单词出现的次数，可能有好几个1，所以要求和
            int sum = 0;
            for (IntWritable v : values)
            {
                //统计单词key出现的次数
                sum += v.get();
            }
            //2.最终统计结果的输出,即<k3,v3>,
            context.write(key, new IntWritable(sum));
        }
    }

    //封装MapReduce作业的所有信息
    public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException
    {
        if (args.length != 2)	// debug only!
        {
            System.out.print("Usage: WordCount <PathToInputFile> <PathToOutput>");
            return;
        }

        //1.创建Job任务
        Configuration conf = new Configuration();

        //创建Job之前，准备清理已经存在的输出目录
        Path outputPath= new Path(args[1]);
        FileSystem fileSystem =FileSystem.get(conf);
        if(fileSystem.exists(outputPath)){
            fileSystem.delete(outputPath,true);
            System.out.println("输出文件夹存在且已被删除");
        }

        //2.创建job，通过getInstance 拿到一个实例
        Job job = Job.getInstance(conf, "word counter");
        //3.指定Jar包位置
        job.setJarByClass(WordCount_final.class);
        //4.关联使用的Mapper类
        job.setMapperClass(WordCountMapper.class);
        //5.关联使用的Reducer类
        job.setReducerClass(WordCountReducer.class);

        //6.设置Mapper阶段输出的数据类型
        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(IntWritable.class);
//		System.out.println("setMapOutputKeyClass: " + Text.class.toString());

        //7.设置Reducer阶段输出的数据类型
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(IntWritable.class);

        //8.设置数据输入的路径
        FileInputFormat.setInputPaths(job, new Path(args[0]));
        //9.设置数据输出的路径
        FileOutputFormat.setOutputPath(job, new Path(args[1]));

        //10.提交任务
        //参数true表示将运行进度等信息及时输出给用户，false的话只是等待作业结束
        boolean rs = job.waitForCompletion(true);
        {
            System.out.println(rs ? "wow, succeed!" : "not so bad");
        }
        System.exit(rs ? 0 : 1);
    }
}
```

然后就可以运行了
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/MapReduce8.png)
<hr>


# 4、将编写的Java项目导出成jar包

![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/MapReduce9.png)
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/MapReduce10.png)
点击OK
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/MapReduce11.png)
点击OK
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/MapReduce12.png)
然后
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/MapReduce13.png)
![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/MapReduce14.png)

完成后就在项目目录下的 out 文件夹下的 artifacts 生成了 jar 包

![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/MapReduce15.png)
<hr>

#  5、将jar包导入到Linux
进入到你的服务器的hadoop所在的文件夹，我的在 /usr/local/hadoop 你们的可能不一样

```bash
cd /usr/local/hadoop
```


在 hadoop 的 bin 文件夹下的 hadoop 文件夹创建文件夹 input

```bash
./bin/hadoop fs -mkdir /input
```

![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/MapReduce16.png)
把 jar 包复制到服务器
移动到创建的文件夹 input 里面
/home/xyj/classes/hadoop.jar 是jar包在我的服务器的位置， 这句话意思是把 jar 包移动到 /input 文件夹



```bash
bin/hadoop fs -put /home/xyj/classes/hadoop.jar /input
```

![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/MapReduce17.png)
查看是否上传成功

```bash
hadoop fs -ls /input
```

![在这里插入图片描述](https://qiniuoss.xuyijie.icu/XuYijieBlog/BlogImage/MapReduce18.png)
输入 `bin/hadoop MapReduce-WordCount.jar wordcount /input /output`运行jar包

MapReduce-WordCount.jar为打包的jar包名，可能你们的不一样

即可获得运行结果


# 总结
没有总结
