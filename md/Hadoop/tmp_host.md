    ## host
192.168.35.111 hadoop-1
192.168.35.113 hadoop-2
192.168.35.112 hadoop-3
192.168.35.120 hadoop-4
192.168.35.121 hadoop-5
192.168.35.122 hadoop-6
192.168.35.123 hadoop-7
    192.168.35.114 dongfeng


    server.4=dongfeng:2888:3888


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
        <value>qjournal://hadoop-3:8485;dongfeng:8485/ns1</value>
    </property>
    <!-- 指定JournalNode在本地磁盘存放数据的位置 -->
    <property>
        <name>dfs.journalnode.edits.dir</name>
        <value>/home/dishui/apps/hadoop-2.4.1/journaldata</value>
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
        <value>/home/dishui/.ssh/id_rsa</value>
    </property>
    <property>
        <name>dfs.ha.fencing.ssh.connect-timeout</name>
        <value>30000</value>
    </property>


<!-- 开启RM高可用 -->
<configuration>
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
       <value>hadoop-1</value>
    </property>
    <property>
       <name>yarn.resourcemanager.hostname.rm2</name>
       <value>hadoop-2</value>
    </property>
    <!-- 指定zk集群地址 -->
    <property>
       <name>yarn.resourcemanager.zk-address</name>
       <value>hadoop-2:2181,hadoop-3:2181,dongfeng:2181</value>
    </property>
    <property>
       <name>yarn.nodemanager.aux-services</name>
       <value>mapreduce_shuffle</value>
    </property>
</configuration>