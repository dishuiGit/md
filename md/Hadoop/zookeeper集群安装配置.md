
[TOC]

### Zookeeper集群搭建
1. 搭建前4台机器配置
    + 修改主机名: `vi /etc/sysconfig/network` (如果已修改,跳过)
    > 主机名快速生效 ==> `sudo hostname dongfeng`
    + 修改主机名 ip映射  `vi /etc/hosts`  
    > `192.168.35.114 dongfeng`
        `192.168.35.111 hadoop-1`
        `192.168.35.113 hadoop-2`
        `192.168.35.112 hadoop-3`

2. 上传zk安装包
    
3. 解压

4. 配置（先在一台节点上配置）
    + 添加一个zoo.cfg配置文件
    > `$ZOOKEEPER/conf`
    `mv zoo_sample.cfg zoo.cfg`
    
    + 修改配置文件（`zoo.cfg`）
    >`dataDir=/home/dishui/zookeeper-3.4.5/data`

    >`server.1=hadoop-1: 2888:3888`
    >`server.2=hadoop-2: 2888:3888`
    >`server.3=hadoop-3: 2888:3888`
    >`server.4=dongfeng: 2888:3888`
    
    + 在（`dataDir=/home/dishui/zookeeper-3.4.5/data`）创建一个`myid`文件，里面内容是`server.N中的N（server.2里面内容为2）`
        `echo "1" > myid`
    
    + 将配置好的zk拷贝到其他节点
        >`scp -r /home/dishui/zookeeper-3.4.5/ hadoop-2:/home/dishui/`
        `scp -r /home/dishui/zookeeper-3.4.5/ hadoop-3:/home/dishui/`
    
    + 注意：在其他节点上一定要修改myid的内容
        >在`hadoop-2`应该讲`myid`的内容改为2 （`echo "2" > myid`）
        >在`hadoop-3`应该讲`myid`的内容改为3 （`echo "3" > myid`）
        
4. 启动集群
    + 分别启动zk
    >`./zkServer.sh start`

    