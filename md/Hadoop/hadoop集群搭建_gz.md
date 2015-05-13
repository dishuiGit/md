
[TOC]

## Hadoop2.0
hadoop2.0已经发布了稳定版本了，增加了很多特性比如HDFS HA、YARN等。最新的hadoop-2.4.1又增加了YARN HA

__注意：__
>apache提供的hadoop-2.4.1的安装包是在32位操作系统编译的，因为hadoop依赖一些C++的本地库，所以如果在64位的操作上安装hadoop-2.4.1
就需要重新在64操作系统上重新编译（建议第一次安装用32位的系统，我将编译好的64位的也上传到群共享里了，如果有兴趣的可以自己编译一下）

### 前期准备
-----------------------
1. 修改Linux主机名
2. 修改IP
3. 修改主机名和IP的映射关系
    __注意:__如果你们公司是租用的服务器或是使用的__云主机__（如华为用主机、阿里云主机等）
    `/etc/hosts`里面要配置的是__内网IP地址__和__主机名__的映射关系    
4. 关闭防火墙
5. ssh免登陆
6. 安装JDK，配置环境变量等

### 集群规划：
    主机名     IP              安装的软件                   运行的进程
    hadoop-1   192.168.1.201   jdk、hadoop               NameNode、DFSZKFailoverController(zkfc)
    hadoop-2   192.168.1.202   jdk、hadoop               NameNode、DFSZKFailoverController(zkfc)
    hadoop-3   192.168.1.203   jdk、hadoop                  ResourceManager
    hadoop-4   192.168.1.204   jdk、hadoop                  ResourceManager
    hadoop-5   192.168.1.205   jdk、hadoop、zookeeper        DataNode、NodeManager、JournalNode、QuorumPeerMain
    hadoop-6   192.168.1.206   jdk、hadoop、zookeeper        DataNode、NodeManager、JournalNode、QuorumPeerMain
    hadoop-7   192.168.1.207   jdk、hadoop、zookeeper        DataNode、NodeManager、JournalNode、QuorumPeerMain
    
------------------------
__说明：__
+ 在`hadoop`2.0中通常由两个`NameNode`组成，一个处于`active`状态，另一个处于`standby`状态。`Active NameNode`对外提供服务，而`Standby NameNode`则不对外提供服务，仅同步`active namenode`的状态，以便能够在它失败时快速进行切换。
+ `hadoop2.0`官方提供了两种`HDFS` `HA`的解决方案，一种是`NFS`，另一种是`QJM`。这里我们使用简单的`QJM`。在该方案中，主备`NameNode`之间通过一组`JournalNode`同步元数据信息，一条数据只要成功写入多数`JournalNode`即认为写入成功。通常配置奇数个`JournalNode`
+ 这里还配置了一个`zookeeper`集群，用于`ZKFC（DFSZKFailoverController`）故障转移，当`Active NameNode`挂掉了，会自动切换`Standby NameNode`为`standby`状态
+ `hadoop-2.2.0`中依然存在一个问题，就是`ResourceManager`只有一个，存在单点故障，`hadoop-2.4.1`解决了这个问题，有两个`ResourceManager`，一个是`Active`，一个是`Standby`，状态由`zookeeper`进行协调

## 安装步骤：
### 安装配置zooekeeper集群（hadoop-5上）
#### 解压
```
    tar -zxvf zookeeper-3.4.5.tar.gz -C /home/hadoop/app/
```
### 修改配置

```
    1. cd /home/hadoop/app/zookeeper-3.4.5/conf/
    2. cp zoo_sample.cfg zoo.cfg
    3. vim zoo.cfg
    4. 修改：dataDir=/home/hadoop/app/zookeeper-3.4.5/tmp
    5. 在最后添加：
        server.1=hadoop-5:2888:3888
        server.2=hadoop-6:2888:3888
        server.3=hadoop-7:2888:3888
    6.保存退出
    7. 然后创建一个tmp文件夹
        mkdir /home/hadoop/app/zookeeper-3.4.5/tmp
    8. 再创建一个空文件
        touch /home/hadoop/app/zookeeper-3.4.5/tmp/myid
    9. 最后向该文件写入ID
        echo 1 > /home/hadoop/app/zookeeper-3.4.5/tmp/myid
```
### 将配置好的zookeeper拷贝到其他节点
```
    首先分别在hadoop-6、hadoop-7根目录下创建一个home/hadoop/app目录：mkdir /home/hadoop/app
        scp -r /home/hadoop/app/zookeeper-3.4.5/ home/hadoop/hadoop-6:/home/hadoop/app/
        scp -r /home/hadoop/app/zookeeper-3.4.5/ home/hadoop/hadoop-7:/home/hadoop/app/
    
    __注意__：修改hadoop-6、hadoop-7对应/home/hadoop/app/zookeeper-3.4.5/tmp/myid内容
    hadoop-6：
        echo 2 > /home/hadoop/app/zookeeper-3.4.5/tmp/myid
    hadoop-7：
        echo 3 > /home/hadoop/app/zookeeper-3.4.5/tmp/myid
```
### 安装配置hadoop集群（在hadoop-1上操作）
#### 解压
```
    tar -zxvf hadoop-2.4.1.tar.gz -C /home/hadoop/app/
```
#### 配置HDFS
>（hadoop2.0所有的配置文件都在$HADOOP_HOME/etc/hadoop目录下）

```
    #将hadoop添加到环境变量中
    vim /etc/profile
    export JAVA_HOME=/usr/java/jdk1.7.0_55
    export HADOOP_HOME=/home/hadoop/app/hadoop-2.4.1
    export PATH=$PATH:$JAVA_HOME/bin:$HADOOP_HOME/bin
    
    #hadoop2.0的配置文件全部在$HADOOP_HOME/etc/hadoop下
    cd /home/hadoop/app/hadoop-2.4.1/etc/hadoop
```
#### 修改`hadoo-env.sh`

```
    export JAVA_HOME=/home/hadoop/app/jdk1.7.0_55
```
#### 修改`core-site.xml`
``` xml
<configuration>
    <!-- 指定hdfs的nameservice为ns1 -->
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://ns1</value>
    </property>
    <!-- 指定hadoop临时目录 -->
    <property>
        <name>hadoop.tmp.dir</name>
        <value>/home/hadoop/app/hadoop-2.4.1/tmp</value>
    </property>

    <!-- 指定zookeeper地址 -->
    <property>
        <name>ha.zookeeper.quorum</name>
        <value>hadoop-5:2181,home/hadoop/hadoop-6:2181,home/hadoop/hadoop-7:2181</value>
    </property>
</configuration>
```

#### 修改`hdfs-site.xml`

```xml
    <configuration>
        <!--指定hdfs的nameservice为ns1，需要和core-site.xml中的保持一致 -->
        <property>
            <name>dfs.nameservices</name>
            <value>ns1</value>
        </property>
        <!-- ns1下面有两个NameNode，分别是nn1，nn2 -->
        <property>
            <name>dfs.ha.namenodes.ns1</name>
            <value>nn1,nn2</value>
        </property>
        <!-- nn1的RPC通信地址 -->
        <property>
            <name>dfs.namenode.rpc-address.ns1.nn1</name>
            <value>hadoop-1:9000</value>
        </property>
        <!-- nn1的http通信地址 -->
        <property>
            <name>dfs.namenode.http-address.ns1.nn1</name>
            <value>hadoop-1:50070</value>
        </property>
        <!-- nn2的RPC通信地址 -->
        <property>
            <name>dfs.namenode.rpc-address.ns1.nn2</name>
            <value>hadoop-2:9000</value>
        </property>
        <!-- nn2的http通信地址 -->
        <property>
            <name>dfs.namenode.http-address.ns1.nn2</name>
            <value>hadoop-2:50070</value>
        </property>
        <!-- 指定NameNode的edits元数据在JournalNode上的存放位置 -->
        <property>
            <name>dfs.namenode.shared.edits.dir</name>
            <value>qjournal://hadoop-5:8485;hadoop-6:8485;hadoop-7:8485/ns1</value>
        </property>
        <!-- 指定JournalNode在本地磁盘存放数据的位置 -->
        <property>
            <name>dfs.journalnode.edits.dir</name>
            <value>/home/hadoop/app/hadoop-2.4.1/journaldata</value>
        </property>
        <!-- 开启NameNode失败自动切换 -->
        <property>
            <name>dfs.ha.automatic-failover.enabled</name>
            <value>true</value>
        </property>
        <!-- 配置失败自动切换实现方式 -->
        <property>
            <name>dfs.client.failover.proxy.provider.ns1</name>
            <value>org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider</value>
        </property>
        <!-- 配置隔离机制方法，多个机制用换行分割，即每个机制暂用一行-->
        <property>
            <name>dfs.ha.fencing.methods</name>
            <value>
                sshfence
                shell(/bin/true)
            </value>
        </property>
        <!-- 使用sshfence隔离机制时需要ssh免登陆 -->
        <property>
            <name>dfs.ha.fencing.ssh.private-key-files</name>
            <value>/home/hadoop/.ssh/id_rsa</value>
        </property>
        <!-- 配置sshfence隔离机制超时时间 -->
        <property>
            <name>dfs.ha.fencing.ssh.connect-timeout</name>
            <value>30000</value>
        </property>
    </configuration>
```
#### 修改`mapred-site.xml`
```xml
    <configuration>
        <!-- 指定mr框架为yarn方式 -->
        <property>
            <name>mapreduce.framework.name</name>
            <value>yarn</value>
        </property>
    </configuration>    
```
#### 修改`yarn-site.xml`
```xml
    <configuration>
            <!-- 开启RM高可用 -->
            <property>
               <name>yarn.resourcemanager.ha.enabled</name>
               <value>true</value>
            </property>
            <!-- 指定RM的cluster id -->
            <property>
               <name>yarn.resourcemanager.cluster-id</name>
               <value>yrc</value>
            </property>
            <!-- 指定RM的名字 -->
            <property>
               <name>yarn.resourcemanager.ha.rm-ids</name>
               <value>rm1,rm2</value>
            </property>
            <!-- 分别指定RM的地址 -->
            <property>
               <name>yarn.resourcemanager.hostname.rm1</name>
               <value>hadoop-3</value>
            </property>
            <property>
               <name>yarn.resourcemanager.hostname.rm2</name>
               <value>hadoop-4</value>
            </property>
            <!-- 指定zk集群地址 -->
            <property>
               <name>yarn.resourcemanager.zk-address</name>
               <value>hadoop-5:2181,hadoop-6:2181,hadoop-7:2181</value>
            </property>
            <property>
               <name>yarn.nodemanager.aux-services</name>
               <value>mapreduce_shuffle</value>
            </property>
    </configuration>
```
    
#### 修改slaves
>(slaves是指定子节点的位置，因为要在hadoop-1上启动HDFS、在hadoop-3启动yarn，所以hadoop-1上的slaves文件指定的是datanode的位置，hadoop-3上的slaves文件指定的是nodemanager的位置)
```
hadoop-5
hadoop-6
hadoop-7
```

#### 配置免密码登陆
+ 首先要配置hadoop-1到hadoop-2、hadoop-3、hadoop-4、hadoop-5、hadoop-6、hadoop-7的免密码登陆在hadoop-1上生产一对钥匙
```bash
ssh-keygen -t rsa
```
+ __将公钥拷贝到其他节点，包括自己__
```bash
ssh-coyp-id hadoop-1
ssh-coyp-id hadoop-2
ssh-coyp-id hadoop-3
ssh-coyp-id hadoop-4
ssh-coyp-id hadoop-5
ssh-coyp-id hadoop-6
ssh-coyp-id hadoop-7
```
+ 配置hadoop-3到hadoop-4、hadoop-5、hadoop-6、hadoop-7的免密码登陆
+ 在hadoop-3上生产一对钥匙
```bash
ssh-keygen -t rsa
```
+ 将公钥拷贝到其他节点
```bash
ssh-coyp-id hadoop-4
ssh-coyp-id hadoop-5
ssh-coyp-id hadoop-6
ssh-coyp-id hadoop-7
```
>注意：两个`namenode`之间要配置ssh免密码登陆，别忘了配置`hadoop-2`到`hadoop-1`的免登陆

+ 在hadoop-2上生产一对钥匙
```bash
ssh-keygen -t rsa
ssh-coyp-id -i hadoop-1                
```

### 将配置好的hadoop拷贝到其他节点
```bash
    scp -r /home/hadoop/app/hadoop-2.4.1 hadoop-2:/home/hadoop/app/
    scp -r /home/hadoop/app/hadoop-2.4.1 hadoop-3:/home/hadoop/app/
    scp -r /home/hadoop/app/hadoop-2.4.1/ hadoop@hadoop-4:/home/hadoop/app/
    scp -r /home/hadoop/app/hadoop-2.4.1/ hadoop@hadoop-5:/home/hadoop/app/
    scp -r /home/hadoop/app/hadoop-2.4.1/ hadoop@hadoop-6:/home/hadoop/app/
    scp -r /home/hadoop/app/hadoop-2.4.1/ hadoop@hadoop-7:/home/hadoop/app/
```
##注意：严格按照下面的步骤
### 启动zookeeper集群
>（分别在hadoop-5、hadoop-6、tcast07上启动zk）
```
    cd /home/hadoop/app/zookeeper-3.4.5/bin/
    ./zkServer.sh start
    #查看状态：一个leader，两个follower
    ./zkServer.sh status
```
### 启动journalnode
>（分别在在hadoop-5、hadoop-6、hadoop-7上执行）
```
    cd /home/hadoop/app/hadoop-2.4.1
    sbin/hadoop-daemon.sh start journalnode
    #运行jps命令检验，hadoop-5、hadoop-6、hadoop-7上多了JournalNode进程
```
### 格式化HDFS
```
    #在hadoop-1上执行命令:
    hdfs namenode -format
    #格式化后会在根据core-site.xml中的hadoop.tmp.dir配置生成个文件，这里我配置的是/home/hadoop/app/hadoop-2.4.1/tmp，然后将/home/hadoop/app/hadoop-2.4.1/tmp拷贝到hadoop-2的/home/hadoop/app/hadoop-2.4.1/下。
    scp -r tmp/ hadoop-2:/home/hadoop/app/hadoop-2.4.1/
    ##也可以这样，在hadoop-2上执行 hdfs namenode -bootstrapStandby
```
### 格式化ZKFC(在hadoop-1上执行即可)
```    
    hdfs zkfc -formatZK
```
### 启动HDFS(在hadoop-1上执行)
```
    sbin/start-dfs.sh
```

### 启动YARN
>(__注意__：是在hadoop-3上执行start-yarn.sh，把namenode和resourcemanager分开是因为性能问题，因为他们都要占用大量资源，所以把他们分开了，他们分开了就要分别在不同的机器上启动)
```
    sbin/start-yarn.sh
```
--------------------

### 检查结果        
>到此，hadoop-2.4.1配置完毕，可以统计浏览器访问:
```
    http://192.168.1.201:50070
    NameNode 'hadoop-1:9000' (active)
    http://192.168.1.202:50070
    NameNode 'hadoop-2:9000' (standby)
```

### 验证HDFS HA
```
首先向hdfs上传一个文件
hadoop fs -put /etc/profile /profile
hadoop fs -ls /
然后再kill掉active的NameNode
kill -9 <pid of NN>
通过浏览器访问：http://192.168.1.202:50070
NameNode 'hadoop-2:9000' (active)
这个时候hadoop-2上的NameNode变成了active
在执行命令：
hadoop fs -ls /
-rw-r--r--   3 root supergroup       1926 2014-02-06 15:36 /profile
刚才上传的文件依然存在！！！
手动启动那个挂掉的NameNode
sbin/hadoop-daemon.sh start namenode
通过浏览器访问：http://192.168.1.201:50070
NameNode 'hadoop-1:9000' (standby)
```
### 验证YARN：
>运行一下hadoop提供的demo中的WordCount程序：
```
hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.4.1.jar wordcount /profile /out
```
__OK，大功告成！！！__

    
            
        
## 测试集群工作状态的一些指令 ：
```
bin/hdfs dfsadmin -report    查看hdfs的各节点状态信息

bin/hdfs haadmin -getServiceState nn1        获取一个namenode节点的HA状态

sbin/hadoop-daemon.sh start namenode  单独启动一个namenode进程

./hadoop-daemon.sh start zkfc   单独启动一个zkfc进程
```
            
##附录1：JPDA 简介
```
Sun Microsystem 的 Java Platform Debugger Architecture (JPDA) 技术是一个多层架构，使您能够在各种环境中轻松调试 Java 应用程序。
JPDA 由两个接口（分别是 JVM Tool Interface 和 JDI）、一个协议（Java Debug Wire Protocol）和两个用于合并它们的软件组件（后端和前端）组成。
它的设计目的是让调试人员在任何环境中都可以进行调试。
更详细的介绍，您可以参考使用 Eclipse 远程调试 Java 应用程序
JDWP 设置
JVM本身就支持远程调试，Eclipse也支持JDWP，只需要在各模块的JVM启动时加载以下参数：

dt_socket表示使用套接字传输。
address=8000
JVM在8000端口上监听请求，这个设定为一个不冲突的端口即可。
server=y
y表示启动的JVM是被调试者。如果为n，则表示启动的JVM是调试器。
suspend=y
y表示启动的JVM会暂停等待，直到调试器连接上才继续执行。suspend=n，则JVM不会暂停等待。

需要在$HADOOP_HOME/etc/hadoop/hadoop-env.sh文件的最后添加你想debug的进程

-远程调试namenode
export HADOOP_NAMENODE_OPTS="-agentlib:jdwp=transport=dt_socket,address=8888,server=y,suspend=y"
-远程调试datanode
export HADOOP_DATANODE_OPTS="-agentlib:jdwp=transport=dt_socket,address=9888,server=y,suspend=y"

-远程调试RM
export YARN_RESOURCEMANAGER_OPTS="-agentlib:jdwp=transport=dt_socket,address=10888,server=y,suspend=y"

-远程调试NM
export YARN_NODEMANAGER_OPTS="-agentlib:jdwp=transport=dt_socket,address=10888,server=y,suspend=y"
```             
##附录2：hadoop datanode节点超时时间设置
```
datanode进程死亡或者网络故障造成datanode无法与namenode通信，
namenode不会立即把该节点判定为死亡，要经过一段时间，这段时间暂称作超时时长。
HDFS默认的超时时长为10分钟+30秒。如果定义超时时间为timeout，则超时时长的计算公式为：
    timeout  = 2 * heartbeat.recheck.interval + 10 * dfs.heartbeat.interval。
    而默认的heartbeat.recheck.interval 大小为5分钟，dfs.heartbeat.interval默认为3秒。
    需要注意的是hdfs-site.xml 配置文件中的
    heartbeat.recheck.interval的单位为毫秒，
    dfs.heartbeat.interval的单位为秒。
    
    所以，举个例子，如果heartbeat.recheck.interval设置为5000（毫秒），dfs.heartbeat.interval设置为3（秒，默认），则总的超时时间为40秒。
    hdfs-site.xml中的参数设置格式：

<property>
        <name>heartbeat.recheck.interval</name>
        <value>2000</value>
</property>
<property>
        <name>dfs.heartbeat.interval</name>
        <value>1</value>
</property>
```          
##附录3： HDFS冗余数据块的自动删除          
```
在日常维护hadoop集群的过程中发现这样一种情况：
    某个节点由于网络故障或者DataNode进程死亡，被NameNode判定为死亡，
    HDFS马上自动开始数据块的容错拷贝；
    当该节点重新添加到集群中时，由于该节点上的数据其实并没有损坏，
    所以造成了HDFS上某些block的备份数超过了设定的备份数。
    通过观察发现，这些多余的数据块经过很长的一段时间才会被完全删除掉，
    那么这个时间取决于什么呢？
    该时间的长短跟数据块报告的间隔时间有关。
    Datanode会定期将当前该结点上所有的BLOCK信息报告给Namenode，
    参数dfs.blockreport.intervalMsec就是控制这个报告间隔的参数。
    
    hdfs-site.xml文件中有一个参数：
<property>
    <name>dfs.blockreport.intervalMsec</name>
    <value>3600000</value>
    <description>Determines block reporting interval in milliseconds.</description>
</property>

    其中3600000为默认设置，3600000毫秒，即1个小时，也就是说，块报告的时间间隔为1个小时，

    所以经过了很长时间这些多余的块才被删除掉。通过实际测试发现，当把该参数调整的稍小一点的时候（60秒），多余的数据块确实很快就被删除了。
```
##附录4：Hadoop机架感知
###1.背景
```
Hadoop在设计时考虑到数据的安全与高效，数据文件默认在HDFS上存放三份，存储策略为本地一份，同机架内其它某一节点上一份，
不同机架的某一节点上一份。这样如果本地数据损坏，节点可以从同一机架内的相邻节点拿到数据，速度肯定比从跨机架节点上拿
数据要快；同时，如果整个机架的网络出现异常，也能保证在其它机架的节点上找到数据。为了降低整体的带宽消耗和读取延时，
HDFS会尽量让读取程序读取离它最近的副本。如果在读取程序的同一个机架上有一个副本，那么就读取该副本。如果一个HDFS集群
跨越多个数据中心，那么客户端也将首先读本地数据中心的副本。那么Hadoop是如何确定任意两个节点是位于同一机架，还是跨机
架的呢？答案就是机架感知。

默认情况下，hadoop的机架感知是没有被启用的。所以，在通常情况下，hadoop集群的HDFS在选机器的时候，是随机选择的，也就是说
，很有可能在写数据时，hadoop将第一块
数据block1写到了rack1上，然后随机的选择下将block2写入到了rack2下，此时两个rack之间产生了数据传输的流量，再接下来，
在随机的情况下，又将block3重新又写回了rack1，此时，两个rack之间又产生了一次数据流量。在job处理的数据量非常的大，
或者往hadoop推送的数据量非常大的时候，这种情况会造成rack之间的网络流量成倍的上升，成为性能的瓶颈，进而影响作业
的性能以至于整个集群的服务
```
###2.配置
```
  默认情况下，namenode启动时候日志是这样的：
2013-09-22 17:27:26,423 INFO org.apache.hadoop.net.NetworkTopology: Adding a new node:  /default-rack/ 192.168.147.92:50010
每个IP 对应的机架ID都是 /default-rack ，说明hadoop的机架感知没有被启用。
要将hadoop机架感知的功能启用，配置非常简单，在 NameNode所在节点的/home/bigdata/apps/hadoop/etc/hadoop的core-site.xml配置文件中配置一个选项:
<property>
  <name>topology.script.file.name</name>
  <value>/home/bigdata/apps/hadoop/etc/hadoop/topology.sh</value>
</property>

这个配置选项的value指定为一个可执行程序，通常为一个脚本，该脚本接受一个参数，输出一个值。接受的参数通常为某台datanode机器的ip地址，
而输出的值通常为该ip地址对应的datanode所在的rack，例如”/rack1”。Namenode启动时，会判断该配置选项是否为空，如果非空，则表示已经启
用机架感知的配置，此时namenode会根据配置寻找该脚本，并在接收到每一个datanode的heartbeat时，将该datanode的ip地址作为参数传给该脚本运行
，并将得到的输出作为该datanode所属的机架ID，保存到内存的一个map中.
至于脚本的编写，就需要将真实的网络拓朴和机架信息了解清楚后，通过该脚本能够将机器的ip地址和机器名正确的映射到相应的机架上去。
一个简单的实现如下：
#!/bin/bash
HADOOP_CONF=/home/bigdata/apps/hadoop/etc/hadoop
while [ $# -gt 0 ] ; do
  nodeArg=$1
  exec<${HADOOP_CONF}/topology.data
  result=""
  while read line ; do
    ar=( $line )
    if [ "${ar[0]}" = "$nodeArg" ]||[ "${ar[1]}" = "$nodeArg" ]; then
      result="${ar[2]}"
    fi
  done
  shift
  if [ -z "$result" ] ; then
    echo -n "/default-rack"
  else
    echo -n "$result"
  fi
  done
topology.data,格式为：节点（ip或主机名） /交换机xx/机架xx
192.168.147.91 tbe192168147091 /dc1/rack1
192.168.147.92 tbe192168147092 /dc1/rack1
192.168.147.93 tbe192168147093 /dc1/rack2
192.168.147.94 tbe192168147094 /dc1/rack3
192.168.147.95 tbe192168147095 /dc1/rack3
192.168.147.96 tbe192168147096 /dc1/rack3
需要注意的是，在Namenode上，该文件中的节点必须使用IP，使用主机名无效，而Jobtracker上，该文件中的节点必须使用主机名，使用IP无效,所以，最好ip和主机名都配上。
这样配置后，namenode启动时候日志是这样的：
2013-09-23 17:16:27,272 INFO org.apache.hadoop.net.NetworkTopology: Adding a new node:  /dc1/rack3/  192.168.147.94:50010
说明hadoop的机架感知已经被启用了。
查看HADOOP机架信息命令:  
./hadoop dfsadmin -printTopology 
Rack: /dc1/rack1
   192.168.147.91:50010 (tbe192168147091)
   192.168.147.92:50010 (tbe192168147092)

Rack: /dc1/rack2
   192.168.147.93:50010 (tbe192168147093)

Rack: /dc1/rack3
   192.168.147.94:50010 (tbe192168147094)
   192.168.147.95:50010 (tbe192168147095)
   192.168.147.96:50010 (tbe192168147096)
```
###3.增加数据节点，不重启NameNode
```
 假设Hadoop集群在192.168.147.68上部署了NameNode和DataNode,启用了机架感知，执行bin/hadoop dfsadmin -printTopology看到的结果：
Rack: /dc1/rack1
   192.168.147.68:50010 (dbj68)
现在想增加一个物理位置在rack2的数据节点192.168.147.69到集群中，不重启NameNode。 
首先，修改NameNode节点的topology.data的配置，加入:192.168.147.69 dbj69 /dc1/rack2,保存。
192.168.147.68 dbj68 /dc1/rack1
192.168.147.69 dbj69 /dc1/rack2
然后，sbin/hadoop-daemons.sh start datanode启动数据节点dbj69,任意节点执行bin/hadoop dfsadmin -printTopology 看到的结果：
Rack: /dc1/rack1
   192.168.147.68:50010 (dbj68)

Rack: /dc1/rack2
   192.168.147.69:50010 (dbj69)
说明hadoop已经感知到了新加入的节点dbj69。 
注意：如果不将dbj69的配置加入到topology.data中，执行sbin/hadoop-daemons.sh start datanode启动数据节点dbj69，datanode日志中会有异常发生，导致dbj69启动不成功。
2013-11-21 10:51:33,502 FATAL org.apache.hadoop.hdfs.server.datanode.DataNode: Initialization failed for block pool Block pool BP-1732631201-192.168.147.68-1385000665316 (storage id DS-878525145-192.168.147.69-50010-1385002292231) service to dbj68/192.168.147.68:9000
org.apache.hadoop.ipc.RemoteException(org.apache.hadoop.net.NetworkTopology$InvalidTopologyException): Invalid network topology. You cannot have a rack and a non-rack node at the same level of the network topology.
  at org.apache.hadoop.net.NetworkTopology.add(NetworkTopology.java:382)
  at org.apache.hadoop.hdfs.server.blockmanagement.DatanodeManager.registerDatanode(DatanodeManager.java:746)
  at org.apache.hadoop.hdfs.server.namenode.FSNamesystem.registerDatanode(FSNamesystem.java:3498)
  at org.apache.hadoop.hdfs.server.namenode.NameNodeRpcServer.registerDatanode(NameNodeRpcServer.java:876)
  at org.apache.hadoop.hdfs.protocolPB.DatanodeProtocolServerSideTranslatorPB.registerDatanode(DatanodeProtocolServerSideTranslatorPB.java:91)
  at org.apache.hadoop.hdfs.protocol.proto.DatanodeProtocolProtos$DatanodeProtocolService$2.callBlockingMethod(DatanodeProtocolProtos.java:20018)
  at org.apache.hadoop.ipc.ProtobufRpcEngine$Server$ProtoBufRpcInvoker.call(ProtobufRpcEngine.java:453)
  at org.apache.hadoop.ipc.RPC$Server.call(RPC.java:1002)
  at org.apache.hadoop.ipc.Server$Handler$1.run(Server.java:1701)
  at org.apache.hadoop.ipc.Server$Handler$1.run(Server.java:1697)
  at java.security.AccessController.doPrivileged(Native Method)
  at javax.security.auth.Subject.doAs(Subject.java:415)
  at org.apache.hadoop.security.UserGroupInformation.doAs(UserGroupInformation.java:1408)
  at org.apache.hadoop.ipc.Server$Handler.run(Server.java:1695)

  at org.apache.hadoop.ipc.Client.call(Client.java:1231)
  at org.apache.hadoop.ipc.ProtobufRpcEngine$Invoker.invoke(ProtobufRpcEngine.java:202)
  at $Proxy10.registerDatanode(Unknown Source)
  at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
  at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:57)
  at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
  at java.lang.reflect.Method.invoke(Method.java:601)
  at org.apache.hadoop.io.retry.RetryInvocationHandler.invokeMethod(RetryInvocationHandler.java:164)
  at org.apache.hadoop.io.retry.RetryInvocationHandler.invoke(RetryInvocationHandler.java:83)
  at $Proxy10.registerDatanode(Unknown Source)
  at org.apache.hadoop.hdfs.protocolPB.DatanodeProtocolClientSideTranslatorPB.registerDatanode(DatanodeProtocolClientSideTranslatorPB.java:149)
  at org.apache.hadoop.hdfs.server.datanode.BPServiceActor.register(BPServiceActor.java:619)
  at org.apache.hadoop.hdfs.server.datanode.BPServiceActor.connectToNNAndHandshake(BPServiceActor.java:221)
  at org.apache.hadoop.hdfs.server.datanode.BPServiceActor.run(BPServiceActor.java:660)
  at java.lang.Thread.run(Thread.java:722)
```
###4.节点间距离计算
```
 有了机架感知，NameNode就可以画出下图所示的datanode网络拓扑图。D1,R1都是交换机，最底层是datanode。
 则H1的rackid=/D1/R1/H1，H1的parent是R1，R1的是D1。这些rackid信息可以通过topology.script.file.name配置。
 有了这些rackid信息就可以计算出任意两台datanode之间的距离，得到最优的存放策略，优化整个集群的网络带宽均衡以及数据最优分配。
distance(/D1/R1/H1,/D1/R1/H1)=0  相同的datanode
distance(/D1/R1/H1,/D1/R1/H2)=2  同一rack下的不同datanode
distance(/D1/R1/H1,/D1/R2/H4)=4  同一IDC下的不同datanode
distance(/D1/R1/H1,/D2/R3/H7)=6  不同IDC下的datanode
```   
            
        
    



 