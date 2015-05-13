[TOC]
##1 HA机制
[![](../../pic/hadoop/81.png)]
[![](../../pic/hadoop/82.png)]
[![](../../pic/hadoop/83.png)]
[![](../../pic/hadoop/84.png)]
##2 Zookeeper
###2.1 Zookeeper介绍
```
什么是Zookeeper？
Zookeeper 是 Google 的 Chubby一个开源的实现，是 Hadoop 的分布式协调服务
它包含一个简单的原语集，分布式应用程序可以基于它实现同步服务，配置维护和命名服务等。
```
```
为什么使用Zookeeper？
大部分分布式应用需要一个主控、协调器或控制器来管理物理分布的子进程（如资源、任务分配等）
目前，大部分应用需要开发私有的协调程序，缺乏一个通用的机制
协调程序的反复编写浪费，且难以形成通用、伸缩性好的协调器
ZooKeeper：提供通用的分布式锁服务，用以协调分布式应用
```
```
Zookeeper能帮我们做什么？
Hadoop2.0,使用Zookeeper的事件处理确保整个集群只有一个活跃的NameNode,存储配置信息等.
HBase,使用Zookeeper的事件处理确保整个集群只有一个HMaster,察觉HRegionServer联机和宕机,存储访问控制列表等.
```
```
Zookeeper的特性：
Zookeeper是简单的
Zookeeper是富有表现力的
Zookeeper具有高可用性
Zookeeper采用松耦合交互方式
Zookeeper是一个资源库
```
```
Zookeeper的数据模型：
Znode有两种类型，短暂的（ephemeral）和持久的（persistent）
Znode的类型在创建时确定并且之后不能再修改
短暂znode的客户端会话结束时，zookeeper会将该短暂znode删除，短暂znode不可以有子节点
持久znode不依赖于客户端会话，只有当客户端明确要删除该持久znode时才会被删除
Znode有四种形式的目录节点，PERSISTENT、PERSISTENT_SEQUENTIAL、EPHEMERAL、EPHEMERAL_SEQUENTIAL
znode 可以是临时节点，一旦创建这个 znode 的客户端与服务器失去联系，这个 znode 也将自动删除，Zookeeper 的客户端和服务器通信采用长连接方式，每个客户端和 服务器通过心跳来保持连接，这个连接状态称为 session，如果 znode 是临时节点，这个 session 失效，znode 也就删除了；持久化目录节点，这个目录节点存储的数据不会丢失；顺序自动编号的目录节点，这种目录节点会根据当前已近存在的节点数自动加 1，然后返回给客户端已经成功创建的目录节点名；临时目录节点，一旦创建这个节点的客户端与服务器端口也就是 session 超时，这种节点会被自动删除；临时自动编号节点
```
```
Zookeeper的角色:
领导者（leader），负责进行投票的发起和决议，更新系统状态
学习者（learner），包括跟随者（follower）和观察者（observer），follower用于接受客户端请求并想客户端返回结果，在选主过程中参与投票
Observer可以接受客户端连接，将写请求转发给leader，但observer不参加投票过程，只同步leader的状态，observer的目的是为了扩展系统，提高读取速度
客户端（client），请求发起方
```
```
Zookeeper的顺序号:
创建znode时设置顺序标识，znode名称后会附加一个值
顺序号是一个单调递增的计数器，由父节点维护
在分布式系统中，顺序号可以被用于为所有的事件进行全局排序，这样客户端可以通过顺序号推断事件的顺序
```
```
Zookeeper的读写机制:
Zookeeper是一个由多个server组成的集群
一个leader，多个follower
每个server保存一份数据副本
全局数据一致
分布式读写
更新请求转发，由leader实施
```
```
Zookeeper的保证:
更新请求顺序进行，来自同一个client的更新请求按其发送顺序依次执行
数据更新原子性，一次数据更新要么成功，要么失败
全局唯一数据视图，client无论连接到哪个server，数据视图都是一致的
实时性，在一定事件范围内，client能读到最新数据
```
```
Zookeeper的API接口:
String create(String path, byte[] data, List<ACL> acl, CreateMode createMode) 
Stat exists(String path, boolean watch) 
void delete(String path, int version) 
List<String> getChildren(String path, boolean watch) 
List<String> getChildren(String path, boolean watch) 
Stat setData(String path, byte[] data, int version) 
byte[] getData(String path, boolean watch, Stat stat) 
void addAuthInfo(String scheme, byte[] auth) 
Stat setACL(String path, List<ACL> acl, int version) 
List<ACL> getACL(String path, Stat stat) 
```
```
观察（watcher）:
Watcher 在 ZooKeeper 是一个核心功能，Watcher 可以监控目录节点的数据变化以及子目录的变化，一旦这些状态发生变化，服务器就会通知所有设置在这个目录节点上的 Watcher，从而每个客户端都很快知道它所关注的目录节点的状态发生变化，而做出相应的反应 
可以设置观察的操作：exists,getChildren,getData
可以触发观察的操作：create,delete,setData
```
```
Zookeeper工作原理:
Zookeeper的核心是原子广播，这个机制保证了各个server之间的同步。实现这个机制的协议叫做Zab协议。Zab协议有两种模式，它们分别是恢复模式和广播模式。当服务启动或者在领导者崩溃后，Zab就进入了恢复模式，当领导者被选举出来，且大多数server的完成了和leader的状态同步以后，恢复模式就结束了。状态同步保证了leader和server具有相同的系统状态。
```
```
Leader选举:
每个Server启动以后都询问其它的Server它要投票给谁。
对于其他server的询问，server每次根据自己的状态都回复自己推荐的leader的id和上一次处理事务的zxid（系统启动时每个server都会推荐自己）
收到所有Server回复以后，就计算出zxid最大的哪个Server，并将这个Server相关信息设置成下一次要投票的Server。
计算这过程中获得票数最多的的sever为获胜者，如果获胜者的票数超过半数，则改server被选为leader。否则，继续这个过程，直到leader被选举出来。
首先看一下选举的过程，zk的实现中用了基于paxos算法（主要是fastpaxos）的实现。具体如下；此外恢复模式下，如果是重新刚从崩溃状态恢复的或者刚启动的的server还会从磁盘快照中恢复数据和会话信息。（zk会记录事务日志并定期进行快照，方便在恢复时进行状态恢复）
leader就会开始等待server连接
Follower连接leader，将最大的zxid发送给leader
Leader根据follower的zxid确定同步点
完成同步后通知follower 已经成为uptodate状态
Follower收到uptodate消息后，又可以重新接受client的请求进行服务了
```
```
总结：
Zookeeper 作为 Hadoop 项目中的一个子项目，是 Hadoop 集群管理的一个必不可少的模块，它主要用来控制集群中的数据，如它管理 Hadoop 集群中的 NameNode，还有 Hbase 中 Master Election、Server 之间状态同步等。
Zoopkeeper 提供了一套很好的分布式集群管理的机制，就是它这种基于层次型的目录树的数据结构，并对树中的节点进行有效管理，从而可以设计出多种多样的分布式的数据管理模型
```
###2.4 zookeeper-dubbo框架机制
[![](../../pic/hadoop/85.png)]
[![](../../pic/hadoop/86.png)]
###2.5 Zookeeper的安装和配置
```
1.上传zk安装包

2.解压

3.配置（先在一台节点上配置）
    3.1添加一个zoo.cfg配置文件
    $ZOOKEEPER/conf
    mv zoo_sample.cfg zoo.cfg
    
    3.2修改配置文件（zoo.cfg）
        dataDir=/itcast/zookeeper-3.4.5/data
        
        server.1=itcast05:2888:3888
        server.2=itcast06:2888:3888
        server.3=itcast07:2888:3888
    
    3.3在（dataDir=/itcast/zookeeper-3.4.5/data）创建一个myid文件，里面内容是server.N中的N（server.2里面内容为2）
        echo "1" > myid
    
    3.4将配置好的zk拷贝到其他节点
        scp -r /itcast/zookeeper-3.4.5/ itcast06:/itcast/
        scp -r /itcast/zookeeper-3.4.5/ itcast07:/itcast/
    
    3.5注意：在其他节点上一定要修改myid的内容
        在itcast06应该讲myid的内容改为2 （echo "6" > myid）
        在itcast07应该讲myid的内容改为3 （echo "7" > myid）
        
4.启动集群
    分别启动zk
        ./zkServer.sh start
    查看状态
        ./zkServer.sh status
    
5. 使用zkServer命令
   ./zkcli.sh 客户端连到本节点
    查看命令帮助  help
    退出 quit
```
