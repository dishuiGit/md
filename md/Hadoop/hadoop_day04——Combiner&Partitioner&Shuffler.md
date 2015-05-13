[TOC]
###1 Combiners编程
```
A 每一个map可能会产生大量的输出，combiner的作用就是在map端对输出先做一次合并，以减少传输到reducer的数据量。
B combiner最基本是实现本地key的归并，combiner具有类似本地的reduce功能。
C 如果不用combiner，那么，所有的结果都是reduce完成，效率会相对低下。使用combiner，先完成的map会在本地聚合，提升速度。
注意：Combiner的输出是Reducer的输入，如果Combiner是可插拔的，添加Combiner绝不能改变最终的计算结果。
所以Combiner只应该用于那种Reduce的输入key/value与输出key/value类型完全一致，且不影响最终结果的场景。比如累加，最大值等。
```
[![](../html/image/60.png)]
[![](../html/image/61.png)]
####1.1改造WordCount
 WordCountMapper 与 WordCountReducer 不变
#####1.1.1WordCountRunner
```
/**
 *  如果要增加combiner组件，就在job中进行设置
 * @author root
 */
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
        
        //设置combiner所使用的类
        job.setCombinerClass(WordCountReducer.class);
        
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
        FileOutputFormat.setOutputPath(job, new Path("hdfs://gaozhe:9000/wordcount/output1"));
        
        //提交本次作业
        job.waitForCompletion(true);
    }
}

```
###2Patitionner编程
```
A Partitioner是partitioner的基类，如果需要定制partitioner也需要继承该类。

B HashPartitioner是mapreduce的默认partitioner。计算方法是
which reducer=(key.hashCode() & Integer.MAX_VALUE) % numReduceTasks，得到当前的目的reducer。

(例子以jar形式运行)
[![](../html/image/HashPartitioner.png)]
```
####2.1改造Flow
```
需求：统计每个手机号的上行流量、下行流量以及总流量
FlowBean、FlowBeanMapper与 FlowBeanReducer 不变
注：执行主类的时候需要传递两个参数（原始数据所在的路径、结果输出路径）
 hadoop jar flow_partithoner.jar com.baidu.flow_patitioner.FlowBeanRunner /flow/sourcedata /flow/output_partithoner
```
#####2.1.1AreaCodePatitioner
```
public class AreaCodePatitioner<KEY, VALUE> extends Partitioner<KEY, VALUE>{

    private static HashMap<String, Integer> areMap= new HashMap<String, Integer>();    
    static{
        areMap.put("137", 0);
        areMap.put("138", 1);
        areMap.put("139", 2);
        areMap.put("135", 3);
        areMap.put("159", 4);
    }

    @Override
    public int getPartition(KEY key, VALUE value, int numPartitions) {
        return areMap.get(key.toString().substring(0, 3))!= null?
                areMap.get(key.toString().substring(0, 3)):5;
    }
}
```
#####2.1.2FlowBeanRunner
```
public class FlowBeanRunner {

    public static void main(String[] args) throws Exception {
        //封装任务信息的对象为Job对象,所以要先构造一个Job对象
        Configuration conf = new Configuration();
        Job job = Job.getInstance(conf);
        
        //设置本次job作业所在的jar包
        job.setJarByClass(FlowBeanRunner.class);
        
        job.setMapperClass(FlowBeanMapper.class);
        job.setReducerClass(FlowBeanReducer.class);
        
        //设置shuffle过程中所使用的partitioner类
        job.setPartitionerClass(AreaCodePatitioner.class);
        
        //设置reducer的数量来跟自定义的partitioner匹配
        //如果task数量超过partition的数量，不报错，但是多余的reducer输出空文件
        //如果task数量少于partition的数量，会报错，因为有一些partition无处可去
        //如果task数量为1，也不会报错，这时getPartition方法失效，因为所有分组都到一个reducer里面去
        job.setNumReduceTasks(6);
        
        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(FlowBean.class);
        
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(FlowBean.class);
           
        //本次job作业要处理的原始数据所在的路径
        FileInputFormat.setInputPaths(job, new Path(args[0]));
        //本次job作业产生的结果输出路径
        FileOutputFormat.setOutputPath(job, new Path(args[1]));
        //提交本次作业
        job.waitForCompletion(true);
    }
}
```
###MapReducer全貌.png
[![](../html/image/MapReducer全貌.png)]
###split切片
####决定map任务数的因素
[![](../html/image/决定map任务数的因素--split.png)]
####split切片信息的源码流程
[![](../html/image/split切片信息的源码流程.png)]
###shuffle机制
[![](../html/image/shuffle机制.png)]
###倒排索引
```
public class InverseIndexStepOne {

    /**
     * 倒排索引第一个步骤所使用mapper
     * @author duanhaitao@itcast.cn
     */
    public static class StepOneMapper extends Mapper<LongWritable, Text, Text, LongWritable>{
        
        @Override
        protected void map(LongWritable key, Text value,Context context)
                throws IOException, InterruptedException {

            //取出一行的内容
            String line = value.toString();
            //切分单词
            String[] words = StringUtils.split(line," ");
            //拿到这一行所属的文件名
            //首先拿到这一行内容所属的切片 
            FileSplit fileSplit = (FileSplit) context.getInputSplit();
            //从切片中获取到文件的路径
            Path filePath = fileSplit.getPath();
            //从路径中获取到文件名
            String fileName = filePath.getName();
            
            //遍历输出
            for(String word:words){
                context.write(new Text(word+"-->"+fileName), new LongWritable(1));
            }
        }
    }
    /**
     * 
     * stpeone所使用reducer
     * @author duanhaitao@itcast.cn
     *
     */
    public static class StepOneReducer extends Reducer<Text, LongWritable, Text, Text>{
        
        @Override
        protected void reduce(Text key, Iterable<LongWritable> values,Context context)
                throws IOException, InterruptedException {
            // key   hello-->a.txt   values: [1,1,1,....]
            long counter =0;
            for(LongWritable value:values){
                counter += value.get();
            }
            
            //key: hello    value:  a.txt-->3
            
            String[] word_fileName = StringUtils.split(key.toString(),"-->");
            String word = word_fileName[0];
            String fileName = word_fileName[1];
            
            //要输出这种结果  key: hello    value:  a.txt-->3
            context.write(new Text(word), new Text(fileName+"-->"+counter));
        }
    }
    
    public static void main(String[] args) throws Exception {
        Configuration conf = new Configuration();
        Job job = Job.getInstance(conf);
        
        job.setJarByClass(InverseIndexStepOne.class);
        
        job.setMapperClass(StepOneMapper.class);
        job.setReducerClass(StepOneReducer.class);
        
    
        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(LongWritable.class);
        
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(Text.class);
        
        FileInputFormat.setInputPaths(job, new Path(args[0]));
        FileOutputFormat.setOutputPath(job, new Path(args[1]));
        
        job.waitForCompletion(true);
    }
}

结果：
hello   a.txt-->3
hello   b.txt-->2
hello   c.txt-->2
jerry   a.txt-->1
jerry   b.txt-->3
jerry   c.txt-->1
tom     a.txt-->2
tom     b.txt-->1
tom     c.txt-->1
```