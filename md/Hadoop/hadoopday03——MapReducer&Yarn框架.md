[TOC]
###MapReduce概述
```
MapReduce是一种分布式计算模型，由Google提出，主要用于搜索领域，解决海量数据的计算问题.
MR由两个阶段组成：Map和Reduce，用户只需要实现map()和reduce()两个函数，即可实现分布式计算，非常简单。
这两个函数的形参是key、value对，表示函数的输入信息。

```
[![](../html/image/MapReduce原理.png)]
###执行步骤：
```
1. map任务处理
1.1 读取输入文件内容，解析成key、value对。对输入文件的每一行，解析成key、value对。每一个键值对调用一次map函数。
1.2 写自己的逻辑，对输入的key、value处理，转换成新的key、value输出。

2.reduce任务处理
2.1 在reduce之前，有一个shuffle的过程对多个map任务的输出进行合并、排序。
2.2写reduce函数自己的逻辑，对输入的key、value处理，转换成新的key、value输出。
2.3 把reduce的输出保存到文件中。
```
###mapreduce编程规范
[![](../html/image/mapreduce编程规范.png)]
###例子1：实现WordCountApp
```
编写代码
打压成jar包
创建文件夹 hadoop fs -mkdir /wordcount/srcdata
创建.txt数据源 hadoop fs -put XX.txt /wordcount/srcdata
运行程序  hadoop jar wordcount.jar  com.baidu.wordcount.WordCountRunner 
查看生成文件夹 hadoop fs -ls /wordcount/output
查看生成文件 hadoop fs -cat /wordcount/output/part-r-00000
```
####mapper
```
public class WordCountMapper extends Mapper<LongWritable, Text, Text, LongWritable>{

    @Override
    protected void map(LongWritable key, Text value,Context context)
            throws IOException, InterruptedException {
        //拿到一行的内容
        String line = value.toString();
        //将这一行切分成单词数组
        String[] words = StringUtils.split(line, " ");
        //遍历单词数组
        for(String word:words){
            //输出<单词，1>这样的key-value对
            context.write(new Text(word), new LongWritable(1));
        }
    }
}
```
####reducer
```
public class WordCountReducer extends Reducer<Text, LongWritable, Text, LongWritable>{

    //调用reduce方法时，传递进来的数据是这样的 ：<hello,{1,1....}>
    @Override
    protected void reduce(Text key, Iterable<LongWritable> values,Context context)
            throws IOException, InterruptedException {

        //定义一个累加计数器
        long counter= 0;
        //遍历values，累加到counter里面
        for(LongWritable value:values){
            
            //累加每一个value
            counter +=value.get();
        }
        // 输出这一个单词的统计结果
        context.write(key, new LongWritable(counter));
    }
}
```
####WordCountRunner
```
public class WordCountRunner {

    public static void main(String[] args) throws Exception {
    
        //封装任务信息的对象为Job对象,所以要先构造一个Job对象
        Configuration conf =new Configuration();
        Job job= Job.getInstance(conf);
        
        //设置本次job作业所在的jar包
        job.setJarByClass(WordCountRunner.class);
        
        //本次job作业使用的mapper类是哪个？
        job.setMapperClass(WordCountMapper.class);
        //本次job作业使用的reducer类是哪个？
        job.setReducerClass(WordCountReducer.class);
        
        //本次job作业mapper类的输出数据key类型
        job.setMapOutputKeyClass(Text.class);
        //本次job作业mapper类的输出数据value类型
        job.setMapOutputValueClass(LongWritable.class);
        
        //本次job作业reducer类的输出数据key类型
        job.setOutputKeyClass(Text.class);
        //本次job作业mapper类的输出数据value类型
        job.setOutputValueClass(LongWritable.class);
        
        //本次job作业要处理的原始数据所在的路径
        FileInputFormat.setInputPaths(job, new Path("hdfs://gaozhe:9000/wordcount/srcdata"));
        //本次job作业产生的结果输出路径
        FileOutputFormat.setOutputPath(job, new Path("hdfs://gaozhe:9000/wordcount/output"));
        
        //提交本次作业
        job.waitForCompletion(true);
    }
}
```
###例子2：统计每个手机号的上行流量、下行流量以及总流量
```
流量求和程序的要点：
1/自定义的数据结构在hadoop中传输需要符合hadoop的序列化规范
        ---- 实现hadoop的一个序列化结构  writable
        ---- 实现其中的两个方法  write()   readField()
        ---- write和read的时候要注意顺序要匹配，就是 先进先出
        ---- hadoop的序列化机制与jdk中的序列化机制有一些区别：
                       ----hadoop的序列化更加精简高效（省略了继承结构），便于在网络中传输
```
####FlowBean
```
public class FlowBean implements Writable{

    private long up_flow;
    private long d_flow;
    private long sum_flow;
    
    //反序列化时需要用到反射机制，所以必须要有一个默认构造方法
    public FlowBean(){}
    
    public FlowBean(long up_flow, long d_flow){
        this.up_flow= up_flow;
        this.d_flow= d_flow;
        this.sum_flow= up_flow + d_flow;
    }
    
    public long getUp_flow() {
        return up_flow;
    }
    public long getD_flow() {
        return d_flow;
    }
    public long getSum_flow() {
        return sum_flow;
    }
    
    /**
     * 序列化，将对象的各个组成部分按byte流顺序写入output流里去
     */
    @Override
    public void write(DataOutput out) throws IOException {
        out.writeLong(up_flow);
        out.writeLong(d_flow);
        out.writeLong(sum_flow);
        //序列化顺序    sum_flow-d_flow-up_flow --->     
    }

    /**
     * 反序列化，从一个数据输入流中按顺序读出对象中的各个组成部分
     * 反序列化时字段的读取顺序一定要与序列化时的顺序匹配
     */
    @Override
    public void readFields(DataInput in) throws IOException {
        // ---> sum_flow-d_flow-up_flow --->
        up_flow = in.readLong();
        d_flow = in.readLong();
        sum_flow = in.readLong();
    }

    @Override
    public String toString() {
        return up_flow + "\t" + d_flow + "\t" + sum_flow;
    }
}

```
####FlowBeanMapper
```
public class FlowBeanMapper extends Mapper<LongWritable, Text, Text, FlowBean>{

    @Override
    protected void map(LongWritable key, Text value,Context context)
            throws IOException, InterruptedException {
        //先拿到一行数据 
        String lineString= value.toString();
        //切分出各个字段
        String[] fileds =lineString.split("\t");
        //拿到手机号字段
        String phoneNum= fileds[1];
        //拿到上行流量字段值
        long up_flow= Long.parseLong(fileds[fileds.length-3]);
        //拿到下行流量字段值
        long d_flow= Long.parseLong(fileds[fileds.length-2]);
        //将上下行流量封装到flowBean中去
        FlowBean flowBean= new FlowBean(up_flow, up_flow);
        //将这个手机号的号码和流量bean输出为一对key-value
        context.write(new Text(phoneNum), flowBean);
    }
}

```
####FlowBeanReducer
```
public class FlowBeanReducer extends Reducer<Text, FlowBean, Text, FlowBean>{

    @Override
    protected void reduce(Text key, Iterable<FlowBean> values,Context context)
            throws IOException, InterruptedException {

        // 拿到的数据：   key：手机号    values: <flowBean,flowBean,flowBean......>
        
        //定义上下行流量的计数器
        long up_flow_counter=0;
        long d_flow_counter=0;
        //循环遍历values进行累加
        for(FlowBean flowBean:values){
            up_flow_counter+= flowBean.getUp_flow();
            d_flow_counter+= flowBean.getD_flow();
        }
        //将统计结果输出
        FlowBean flowBean= new FlowBean(up_flow_counter, d_flow_counter);
        context.write(key, flowBean);
    }
}

```
####FlowBeanRunner
```
package com.baidu.flow;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

public class FlowBeanRunner {

    public static void main(String[] args) throws Exception {
        //封装任务信息的对象为Job对象,所以要先构造一个Job对象
        Configuration conf = new Configuration();
        Job job = Job.getInstance(conf);
        
        //设置本次job作业所在的jar包
        job.setJarByClass(FlowBeanRunner.class);
        
        job.setMapperClass(FlowBeanMapper.class);
        job.setReducerClass(FlowBeanReducer.class);
        
        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(FlowBean.class);
        
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(FlowBean.class);
        
        FileInputFormat.setInputPaths(job, new Path("hdfs://gaozhe:9000/flow/sourcedata"));
        FileOutputFormat.setOutputPath(job, new Path("hdfs://gaozhe:9000/flow/output"));
        //提交本次作业
        job.waitForCompletion(true);
    }
}

```
结果：
[![](../html/image/流量统计.png)]
###Yarn框架技术机制
[![](../html/image/50.png)]
[![](../html/image/51.png)]
[![](../html/image/52.png)]