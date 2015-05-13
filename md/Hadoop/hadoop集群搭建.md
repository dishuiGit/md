
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
            
                
            
            
    
            
        
    



 