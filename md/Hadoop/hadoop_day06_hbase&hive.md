[TOC]
#HBase
##1 HBase简介
```
HBase – Hadoop Database，是一个高可靠性、高性能、面向列、可伸缩的分布式存储系统，利用HBase技术可在廉价PC Server上搭建起大规模结构化存储集群。
HBase利用Hadoop HDFS作为其文件存储系统，利用Hadoop MapReduce来处理HBase中的海量数据，利用Zookeeper作为协调工具。
```
##2 Hbase表结构
[![](../../pic/hadoop/hbase表结构.png)]
###主键：Row Key 
```
主键是用来检索记录的主键，访问hbase table中的行，只有三种方式
通过单个row key访问
通过row key的range
全表扫描
```
###列族：Column Family
```
列族在创建表的时候声明，一个列族可以包含多个列，列中的数据都是以二进制形式存在，没有数据类型。
```
###时间戳：timestamp
```
HBase中通过row和columns确定的为一个存贮单元称为cell。每个 cell都保存着同一份数据的多个版本。版本通过时间戳来索引
```
##3 HBASE基础知识
###架构体系
```
Client  
    包含访问hbase 的接口，client 维护着一些cache 来加快对hbase 的访问，比如regione 的位置信息

Zookeeper
    保证任何时候，集群中只有一个running master
    存贮所有Region 的寻址入口
    实时监控RegionServer的状态，将Regionserver的上线和下线信息，实时通知给Master
    存储Hbase 的schema,包括有哪些table，每个table 有哪些column family

Master可以启动多个HMaster，通过Zookeeper的MasterElection机制保证总有一个Master运行
    为Region server 分配region
    负责region server 的负载均衡
    发现失效的region server 并重新分配其上的region
```
###Region Server
```
维护Master 分配给它的region，处理对这些region 的IO 请求
负责切分在运行过程中变得过大的region

可以看出，client 访问hbase 上数据的过程并不需要master 参与，寻址访问先zookeeper再regionserver，数据读写访问regioneserver。
HRegionServer主要负责响应用户I/O请求，向HDFS文件系统中读写数据，是HBase中最核心的模块。    
```
##4 Hbase集群搭建
```
1.上传hbase安装包

2.解压

3.配置hbase集群，要修改3个文件（首先zk集群已经安装好了  HMASTER<主>  REGIONSERVER<从>）
    注意：要把hadoop的hdfs-site.xml和core-site.xml 放到hbase/conf下
    
    3.1修改hbase-env.sh
    export JAVA_HOME=/usr/java/jdk1.7.0_55
    //告诉hbase使用外部的zk
    export HBASE_MANAGES_ZK=false
    
    vim hbase-site.xml
    <configuration>
        <!-- 指定hbase在HDFS上存储的路径 -->
        <property>
                <name>hbase.rootdir</name>
                <value>hdfs://ns1/hbase</value>
        </property>
        <!-- 指定hbase是分布式的 -->
        <property>
                <name>hbase.cluster.distributed</name>
                <value>true</value>
        </property>
        <!-- 指定zk的地址，多个用“,”分割 -->
        <property>
                <name>hbase.zookeeper.quorum</name>
                <value>weekend05:2181,weekend06:2181,weekend07:2181</value>
        </property>
    </configuration>
    
    vim regionservers
    weekend03
    weekend04
    weekend05
    weekend06
    
    3.2拷贝hbase到其他节点
        scp -r /weekend/hbase-0.96.2-hadoop2/ weekend02:/weekend/
        scp -r /weekend/hbase-0.96.2-hadoop2/ weekend03:/weekend/
        scp -r /weekend/hbase-0.96.2-hadoop2/ weekend04:/weekend/
        scp -r /weekend/hbase-0.96.2-hadoop2/ weekend05:/weekend/
        scp -r /weekend/hbase-0.96.2-hadoop2/ weekend06:/weekend/

4.将配置好的HBase拷贝到每一个节点并同步时间。

5.启动所有的hbase
    分别启动zk
        ./zkServer.sh start
    启动hbase集群
        start-dfs.sh
    启动hbase，在主节点上运行：
        start-hbase.sh
6.通过浏览器访问hbase管理页面
    192.168.1.201:60010

7.为保证集群的可靠性，要启动多个HMaster
    hbase-daemon.sh start master
```
##5 Hbase shell操作
```
进入hbase命令行
./hbase shell

显示hbase中的表
list

创建user表，包含info、data两个列族
create 'user', 'info1', 'data1'
create 'user', {NAME => 'info', VERSIONS => '3'}

向user表中插入信息，row key为rk0001，列族info中添加name列标示符，值为zhangsan
put 'user', 'rk0001', 'info:name', 'zhangsan'

向user表中插入信息，row key为rk0001，列族info中添加gender列标示符，值为female
put 'user', 'rk0001', 'info:gender', 'female'

向user表中插入信息，row key为rk0001，列族info中添加age列标示符，值为20
put 'user', 'rk0001', 'info:age', 20

向user表中插入信息，row key为rk0001，列族data中添加pic列标示符，值为picture
put 'user', 'rk0001', 'data:pic', 'picture'

获取user表中row key为rk0001的所有信息
get 'user', 'rk0001'

获取user表中row key为rk0001，info列族的所有信息
get 'user', 'rk0001', 'info'

获取user表中row key为rk0001，info列族的name、age列标示符的信息
get 'user', 'rk0001', 'info:name', 'info:age'

获取user表中row key为rk0001，info、data列族的信息
get 'user', 'rk0001', 'info', 'data'
get 'user', 'rk0001', {COLUMN => ['info', 'data']}

get 'user', 'rk0001', {COLUMN => ['info:name', 'data:pic']}

获取user表中row key为rk0001，列族为info，版本号最新5个的信息
get 'user', 'rk0001', {COLUMN => 'info', VERSIONS => 2}
get 'user', 'rk0001', {COLUMN => 'info:name', VERSIONS => 5}
get 'user', 'rk0001', {COLUMN => 'info:name', VERSIONS => 5, TIMERANGE => [1392368783980, 1392380169184]}

获取user表中row key为rk0001，cell的值为zhangsan的信息
get 'people', 'rk0001', {FILTER => "ValueFilter(=, 'binary:图片')"}

获取user表中row key为rk0001，列标示符中含有a的信息
get 'people', 'rk0001', {FILTER => "(QualifierFilter(=,'substring:a'))"}

put 'user', 'rk0002', 'info:name', 'fanbingbing'
put 'user', 'rk0002', 'info:gender', 'female'
put 'user', 'rk0002', 'info:nationality', '中国'
get 'user', 'rk0002', {FILTER => "ValueFilter(=, 'binary:中国')"}


查询user表中的所有信息
scan 'user'

查询user表中列族为info的信息
scan 'user', {COLUMNS => 'info'}
scan 'user', {COLUMNS => 'info', RAW => true, VERSIONS => 5}
scan 'persion', {COLUMNS => 'info', RAW => true, VERSIONS => 3}
查询user表中列族为info和data的信息
scan 'user', {COLUMNS => ['info', 'data']}
scan 'user', {COLUMNS => ['info:name', 'data:pic']}


查询user表中列族为info、列标示符为name的信息
scan 'user', {COLUMNS => 'info:name'}

查询user表中列族为info、列标示符为name的信息,并且版本最新的5个
scan 'user', {COLUMNS => 'info:name', VERSIONS => 5}

查询user表中列族为info和data且列标示符中含有a字符的信息
scan 'user', {COLUMNS => ['info', 'data'], FILTER => "(QualifierFilter(=,'substring:a'))"}

查询user表中列族为info，rk范围是[rk0001, rk0003)的数据
scan 'people', {COLUMNS => 'info', STARTROW => 'rk0001', ENDROW => 'rk0003'}

查询user表中row key以rk字符开头的
scan 'user',{FILTER=>"PrefixFilter('rk')"}

查询user表中指定范围的数据
scan 'user', {TIMERANGE => [1392368783980, 1392380169184]}

删除数据
删除user表row key为rk0001，列标示符为info:name的数据
delete 'people', 'rk0001', 'info:name'
删除user表row key为rk0001，列标示符为info:name，timestamp为1392383705316的数据
delete 'user', 'rk0001', 'info:name', 1392383705316


清空user表中的数据
truncate 'people'


修改表结构
首先停用user表（新版本不用）
disable 'user'

添加两个列族f1和f2
alter 'people', NAME => 'f1'
alter 'user', NAME => 'f2'
启用表
enable 'user'


###disable 'user'(新版本不用)
删除一个列族：
alter 'user', NAME => 'f1', METHOD => 'delete' 或 alter 'user', 'delete' => 'f1'

添加列族f1同时删除列族f2
alter 'user', {NAME => 'f1'}, {NAME => 'f2', METHOD => 'delete'}

将user表的f1列族版本号改为5
alter 'people', NAME => 'info', VERSIONS => 5
启用表
enable 'user'


删除表
disable 'user'
drop 'user'


get 'person', 'rk0001', {FILTER => "ValueFilter(=, 'binary:中国')"}
get 'person', 'rk0001', {FILTER => "(QualifierFilter(=,'substring:a'))"}
scan 'person', {COLUMNS => 'info:name'}
scan 'person', {COLUMNS => ['info', 'data'], FILTER => "(QualifierFilter(=,'substring:a'))"}
scan 'person', {COLUMNS => 'info', STARTROW => 'rk0001', ENDROW => 'rk0003'}

scan 'person', {COLUMNS => 'info', STARTROW => '20140201', ENDROW => '20140301'}
scan 'person', {COLUMNS => 'info:name', TIMERANGE => [1395978233636, 1395987769587]}
delete 'person', 'rk0001', 'info:name'

alter 'person', NAME => 'ffff'
alter 'person', NAME => 'info', VERSIONS => 10


get 'user', 'rk0002', {COLUMN => ['info:name', 'data:pic']}
```
##6 Hbase API操作
```
package cn.itcast.hbase;

public class HbaseDemo {

    private Configuration conf = null;
    
    @Before
    public void init(){
        conf = HBaseConfiguration.create();
        conf.set("hbase.zookeeper.quorum", "hadoop01,hadoop02,hadoop03");
    }
    
    /**
     * 删除一个表
     */
    @Test
    public void testDrop() throws Exception{
        HBaseAdmin admin = new HBaseAdmin(conf);
        admin.disableTable("account");
        admin.deleteTable("account");
        admin.close();
    }
    
    /**
     * 往表中插入数据
     */
    @Test
    public void testPut() throws Exception{
        HTable table = new HTable(conf, "user");
        //hbase表只支持byte[]类型
        Put put = new Put(Bytes.toBytes("rk0003"));
        put.add(Bytes.toBytes("info"), Bytes.toBytes("name"), Bytes.toBytes("liuyan"));
        table.put(put);
        table.close();
    }
    
    /**
     * 查询数据
     * get 是一次只能产讯一行的数据
     */
    @Test
    public void testGet() throws Exception{
        //HTablePool pool = new HTablePool(conf, 10);
        //HTable table = (HTable) pool.getTable("user");
        HTable table = new HTable(conf, "user");
        Get get = new Get(Bytes.toBytes("rk0001"));
        //get.addColumn(Bytes.toBytes("info"), Bytes.toBytes("name"));
        get.setMaxVersions(5);
        Result result = table.get(get);
        //result.getValue(family, qualifier)
        for(KeyValue kv : result.list()){
            String family = new String(kv.getFamily());
            System.out.println(family);
            String qualifier = new String(kv.getQualifier());
            System.out.println(qualifier);
            System.out.println(new String(kv.getValue()));
        }
        table.close();
    }
    
    /**
     * 多种过滤条件的使用方法
     */
    @Test
    public void testScan() throws Exception{
        HTablePool pool = new HTablePool(conf, 10);
        HTableInterface table = pool.getTable("user");

        Scan scan = new Scan(Bytes.toBytes("rk0001"), Bytes.toBytes("rk0002"));
        scan.addFamily(Bytes.toBytes("info"));
        //scan的查询入口
        ResultScanner scanner = table.getScanner(scan);
        for(Result r : scanner){
            /**
            for(KeyValue kv : r.list()){
                String family = new String(kv.getFamily());
                System.out.println(family);
                String qualifier = new String(kv.getQualifier());
                System.out.println(qualifier);
                System.out.println(new String(kv.getValue()));
            }
            */
            byte[] value = r.getValue(Bytes.toBytes("info"), Bytes.toBytes("name"));
            System.out.println(new String(value));
        }
        pool.close();
    }

    /**
     * 从表中删除数据
     */
    @Test
    public void testDel() throws Exception{
        HTable table = new HTable(conf, "user");
        Delete del = new Delete(Bytes.toBytes("rk0001"));
        del.deleteColumn(Bytes.toBytes("data"), Bytes.toBytes("pic"));
        table.delete(del);
        table.close();
    }
    
    /**
     * 创建表
     */
    public static void main(String[] args) throws Exception {
        //2 构建一个配置器
        Configuration conf = HBaseConfiguration.create();
        conf.set("hbase.zookeeper.quorum", "hadoop01,hadoop02,hadoop03");
        
        //1 拿到建表的工具类
        HBaseAdmin admin = new HBaseAdmin(conf);
        //5 创建一个TableName
        TableName name= TableNameOf("mybabies");
        //4 创建表的描述器,发现一个TableName
        HTableDescriptor desc = new HTableDescriptor("name");

        //7 创建列族的描述器
        HColumnDescriptor familyBaseinfo =new HColumnDescriptor("base_info");
        familyBaseinfo.setMaxVersions(3);
        HColumnDescriptor familyBaseinfo =new HColumnDescriptor("extra_info");

        //6 应该往表描述器中添加一些列族，发现列族是通过列族描述器来传
        desc.addFamily(familyBaseinfo);
        desc.addFamily(familyBaseinfo);

        //3 用这个admin去建表，发现要一个表的描述器
        admin.createTable(desc);

        //8 表创建完成，关闭连接
        admin.close();
    }
    
    public void createTable(String tableName, int maxVersion, String... cf){
    }
}
```
===========================================================================
#hive
##1 引入Hive（背景）
用MapReduce进行联表查询
```
商品信息                 订单信息
a.txt                     b.txt

1,apple                   1,10010
2,orange                  2,10011
3,banana                  3,10012
4,potato                  3,10013
                          2,10016
map输出
<3,banana-->a.txt>        <3,[10012-->b.txt,10013-->b.txt]>

到达reduce的时候是这样的数据
<3,[banana-->a.txt,10012-->b.txt,10013-->b.txt,10014-->b.txt]>

reduce中的逻辑
就是把来自于a.txt的value跟其他的values一个一个地去拼接，然后输出

select a.name,b.order_nbr from a,b where a.id=b.id

想要的结果：
apple,10010
orange,10011
banana,10012
banana,10013
orange,100016

结论：表多且数据量大的时候，手动写MapReduce工作流很大
```
##2 什么是Hive
```
Hive 是建立在 Hadoop上的数据仓库基础构架。它提供了一系列的工具，可以用来进行数据提取转化加载（ETL ），
这是一种可以存储、查询和分析存储在 Hadoop  中的大规模数据的机制。Hive 定义了简单的类 SQL  查询语言，称为 QL ，
它允许熟悉 SQL  的用户查询数据。同时，这个语言也允许熟悉 MapReduce  开发者的开发自定义的 mapper和 reducer来
处理内建的 mapper和reducer  无法完成的复杂的分析工作。
总结： 1 可以把文件转换成表 
      2 用类sql语句进行查询
      3 支持自定义MapReduce 

Hive是SQL解析引擎，它将SQL语句转译成M/R Job然后在Hadoop执行。
Hive的表其实就是HDFS的目录/文件，按表名把文件夹分开。如果是分区表，则分区值是子文件夹，可以直接在M/R Job里使用这些数据。
```
##3 Hive的系统架构
```
用户接口主要有三个：CLI，JDBC/ODBC和 WebUI
a: CLI，即Shell命令行
b: JDBC/ODBC 是 Hive 的Java，与使用传统数据库JDBC的方式类似
c: WebGUI是通过浏览器访问 Hive

Hive 将元数据存储在数据库中(metastore)，目前只支持 mysql、derby。Hive 中的元数据包括表的名字，表的列和分区及其属性，表的属性（是否为外部表等），表的数据所在目录等

解释器、编译器、优化器完成 HQL 查询语句从词法分析、语法分析、编译、优化以及查询计划（plan）的生成。生成的查询计划存储在 HDFS 中，并在随后有 MapReduce 调用执行

Hive 的数据存储在 HDFS 中，大部分的查询由 MapReduce 完成（包含 * 的查询，比如 select * from table 不会生成 MapRedcue 任务
```
##4 hive的安装
Hive只在一个节点上安装即可

###1.上传tar包
```
    hive-0.9.0.tar.gz
```
###2.解压
```
    tar -zxvf hive-0.9.0.tar.gz -C /cloud/
```    
###3.配置mysql metastore（切换到root用户）
```
    配置HIVE_HOME环境变量
    rpm -qa | grep mysql
    rpm -e mysql-libs-5.1.66-2.el6_3.i686 --nodeps
    rpm -ivh MySQL-server-5.1.73-1.glibc23.i386.rpm 
    rpm -ivh MySQL-client-5.1.73-1.glibc23.i386.rpm 
    修改mysql的密码
    /usr/bin/mysql_secure_installation
    （注意：删除匿名用户，允许用户远程连接）
    service mysqld start 开启msql服务
    登陆mysql
    mysql -u root -p

    在默认的情况下MySQL没有密码，我们要自己进行设置，设置方法如下：

 键入以下命令：
[root@test1 local]# /usr/bin/mysqladmin -u root password 123456 
要换密码的话用下条命令：

格式：mysqladmin -u 用户名 -p 旧密码 password 新密码

修改后的密码登录
[root@mail ~]# mysql -u root -p 

查看数据库的命令是
show databases；（最后有个分号）
```
###4.配置hive
```
    cp hive-default.xml.template hive-site.xml 
    修改hive-site.xml（删除所有内容，只留一个<property></property>）
    添加如下内容：
    <property>
      <name>javax.jdo.option.ConnectionURL</name>
      <value>jdbc:mysql://hadoop00:3306/hive?createDatabaseIfNotExist=true</value>
      <description>JDBC connect string for a JDBC metastore</description>
    </property>

    <property>
      <name>javax.jdo.option.ConnectionDriverName</name>
      <value>com.mysql.jdbc.Driver</value>
      <description>Driver class name for a JDBC metastore</description>
    </property>

    <property>
      <name>javax.jdo.option.ConnectionUserName</name>
      <value>root</value>
      <description>username to use against metastore database</description>
    </property>

    <property>
      <name>javax.jdo.option.ConnectionPassword</name>
      <value>123</value>
      <description>password to use against metastore database</description>
    </property>
```   
###5.安装hive和mysq完成后，将mysql的连接jar包拷贝到$HIVE_HOME/lib目录下
```
    如果出现没有权限的问题，在mysql授权(在安装mysql的机器上执行)
    mysql -uroot -p
    #(执行下面的语句  *.*:所有库下的所有表   %：任何IP地址或主机都可以连接)
    GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '123' WITH GRANT OPTION;
    FLUSH PRIVILEGES;
```   
###6.建表(默认是内部表)
导入，文本文件的字段多了不管，少了表中补null
[![](../../pic/hadoop/123.png)]
hive>select count(*) from xqx;
```
    bin目录下>./hive
    create table trade_detail(id bigint, account string, income double, expenses double, time string) row format delimited fields terminated by '\t';
    建分区表p
    create table td_part(id bigint, account string, income double, expenses double, time string) partitioned by (logdate string) row format delimited fields terminated by '\t';
    建外部表
    create external table td_ext(id bigint, account string, income double, expenses double, time string) row format delimited fields terminated by '\t' location '/td_ext';
```
###7.创建分区表
```
    普通表和分区表区别：有大量数据增加的需要建分区表
    create table book (id bigint, name string) partitioned by (pubdate string) row format delimited fields terminated by '\t'; 

    分区表加载数据
    load data local inpath './book.txt' overwrite into table book partition (pubdate='2010-08-22');
    
    load data local inpath '/root/data.am' into table beauty partition (nation="USA");

    select nation, avg(size) from beauties group by nation order by avg(size);
```    
##5 hive的工作机制
```
1、在hive中建一个库
    ---在hive的元数据库中记录
    ---在hdfs的默认路径下/user/hive/warehouse/ 建一个以 "库名.db" 为名字的文件夹

2、在hive的库中建表
        ---在hive的元数据库中记录
        ---在hdfs的默认路径下  /user/hive/warehouse/库.db/  下建一个 “表名” 为名字的文件夹  
        
3、hive中内部表和外部表的区别

    ----建表时，内部表不用指定数据存放的路径，默认都放在        /user/hive/warehouse/
    ----外部表建表时，要指定external关键字，同时要指定数据存放的路径（要分析的数据在哪就指定哪）
    
    ----内部表删除时，会清掉元数据，同时删掉表文件夹及其中的数据
    ----外部表删除时，只清除元数据
    
    
4、hive表的数据可以存成多种文件格式，最普通的是textfile，但是性能比较好的是 sequenceFile格式
    ----sequencefile 是一种二进制文件
    ----文件内的内容组织形式为key:value
    ----在hadoop有一个优化场景可以使用sequencefile
             小文件合并成大文件：  
                  ---读一个小文件，就把小文件的文件名作为key，内容作为value，追加到一个大sequencefile文件中
    ----sequencefile文件格式支持较好的压缩性能，而且hadoop的mapreduce程序可以直接从sequencefile的压缩文件中直接读取数据
                
5、在linux的shell中直接运行HQL语句的方法
//cli shell  
hive -S -e 'select country,count(*) from tab_ext' > /home/hadoop/hivetemp/e.txt 

这种运行机制非常重要，在生产中就是用这种机制来将大量的HQL逻辑组织在一个批量执行的shell脚本程序中

6、分区表
分区表的意义在于可以针对一个分区来进行统计从而减小统计的数据集
创建分区表要使用关键字  partitioned by (country string)
导入数据到分区表的时候需要指定这份数据所属的分区     load data ..... partition(country='china') 
hive就会在hdfs的表目录建一个分区子文件夹，名字为 country=china ,这一个分区的数据就放在该子文件夹下
针对分区进行的查询和统计只要指定 where条件，将分区标识看成一个普通表字段就可以  where country='china'

7、在hive中自定义函数
----首先自定义一个java类来实现这个函数的业务逻辑
```

