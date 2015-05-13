[TOC]
##1、ssh免密登陆的配置方法：

###方法一：ssh-keygen -t rsa 在客户端生成密钥对
   把公钥拷贝给要登录的目标主机，
   目标主机上将这个公钥加入到授权列表  cat id_rsa.pub >> authorized_keys
   目标主机还要将这个授权列表文件权限修改为600 chmod 600 authorized_keys 
   
###方法二：只需要在客户端上操作就可以
	 ssh-keygen -t rsa 在客户端生成密钥对
	 ssh-copy-id B主机
##2、hdfs的shell操作
```
调用文件系统(FS)Shell命令应使用 bin/hadoop fs 的形式。
所有的FS shell命令使用URI路径作为参数。
　URI格式是scheme://authority/path。HDFS的scheme是hdfs，对本地文件系统，scheme是file。其中scheme和authority参数都是

可选的，如果未加指定，就会使用配置中指定的默认scheme。
　例如：/parent/child可以表示成hdfs://namenode:namenodePort/parent/child，或者更简单的/parent/child（假设配置文件是

namenode:namenodePort）
大多数FS Shell命令的行为和对应的Unix Shell命令类似。


--查看所有操作命令
hadoop fs 不要带其他参数，就会列出所有的可用参数

--查看某个hdfs的路径下的文件或文件夹体积大小
hadoop fs -du -s -h hdfs://yun-11:9000/*

-help [cmd]	//显示命令的帮助信息
-ls(r) <path>	//显示当前目录下所有文件
-du(s) <path>	//显示目录中所有文件大小
-count[-q] <path>	//显示目录中文件数量
-mv <src> <dst>	//移动多个文件到目标目录
-cp <src> <dst>	//复制多个文件到目标目录
-rm(r)		//删除文件(夹)
-put <localsrc> <dst>	//本地文件复制到hdfs
-copyFromLocal	//同put
-moveFromLocal	//从本地文件移动到hdfs
-get [-ignoreCrc] <src> <localdst>	//复制文件到本地，可以忽略crc校验
-getmerge <src> <localdst>		//将源目录中的所有文件排序合并到一个文件中
-cat <src>	//在终端显示文件内容
-text <src>	//在终端显示文件内容
-copyToLocal [-ignoreCrc] <src> <localdst>	//复制到本地
-moveToLocal <src> <localdst>
-mkdir <path>	//创建文件夹
-touchz <path>	//创建一个空文件
```
##3、HDFS架构
[![](../html/image/31.png)]
###3.1 namenode
是整个文件系统的管理节点。它维护着整个文件系统的文件目录树，文件/目录的元信息和每个文件对应的数据块列表。接收用户的操作请求。
文件包括：
fsimage:元数据镜像文件。存储某一时段NameNode内存元数据信息。
edits:操作日志文件。
fstime:保存最近一次checkpoint的时间
以上这些文件是保存在linux的文件系统中。

###3.2 NameNode的工作特点
Namenode始终在内存中保存metedata，用于处理“读请求”
到有“写请求”到来时，namenode会首先写editlog到磁盘，即向edits文件中写日志，成功返回后，才会修改内存，并且向客户端返回
Hadoop会维护一个fsimage文件，也就是namenode中metedata的镜像，但是fsimage不会随时与namenode内存中的metedata保持一致，而是每隔一段时间通过合并edits文件来更新内容。Secondary namenode就是用来合并fsimage和edits文件来更新NameNode的metedata的。

###3.3 namenode管理元数据的一些细节
---职责：响应客户端请求  维护hdfs的目录树  管理元数据（hdfs上的文件---blocks---datanode）
---元数据在namenode上的存在位置有三个：一个在内存中，一个在磁盘的fsimage文件中，一个在edits文件中
---新增元数据： 客户端往hdfs上传数据时，namenode先在edits中记录元信息，如果客户端上传成功，就会在内存的元数据中增加一条元数据
---checkpoint： 当满足触发条件时，secondarynamenode会从namenode上下载edits和
fsimage这两个文件，然后在本地合

并好，再上传到namenode上替换掉老的fsimage

###3.4 SecondaryNameNode
HA的一个解决方案。但不支持热备。配置即可。
执行过程：从NameNode上下载元数据信息（fsimage,edits），然后把二者合并，生成新的fsimage，在本地保存，并将其推送到NameNode，替换旧的fsimage.
默认在安装在NameNode节点上，但这样...不安全！

###3.5 secondary namenode的工作流程
secondary通知namenode切换edits文件
secondary从namenode获得fsimage和edits(通过http)
secondary将fsimage载入内存，然后开始合并edits
secondary将新的fsimage发回给namenode
namenode用新的fsimage替换旧的fsimage

###3.6 什么时候checkpiont 
fs.checkpoint.period 指定两次checkpoint的最大时间间隔，默认3600秒。 
fs.checkpoint.size    规定edits文件的最大值，一旦超过这个值则强制checkpoint，不管是否到达最大时间间隔。默认大小是64M。
[![](../html/image/32.png)]

###3.7 Datanode
提供真实文件数据的存储服务。
文件块（block）：最基本的存储单位。对于文件内容而言，一个文件的长度大小是size，那么从文件的０偏移开始，按照固定的大小，顺序对文件进行划分并编号，划分好的每一个块称一个Block。HDFS默认Block大小是128MB，以一个256MB文件，共有256/128=2个Block.
不同于普通文件系统的是，HDFS中，如果一个文件小于一个数据块的大小，并不占用整个数据块存储空间
Replication。多复本。默认是三个。

###3.8、datanode存储block的一些细节
```
--- 职责：提供客户文件block块的读写
--- 细节：block块有大小的约束（配置文件中，存在一个优先级的问题）
          datanode还会定期向namenode汇报自己所管理的block的信息
          block块存在datanode所在的本地磁盘，目录是配置文件中指定的dfs.data.dir
          


tips:   在部署、使用hadoop的过程中hdfs容易出现的几个问题
        1、主机名的问题，在配置文件中，关于写主机名的地方，不要用下划线，也不要用localhost，也不要用ip
		2、如果出现datanode启动失败的情况，可以通过查看logs下的datanode的日志来分析clusterID不一致问题（常常是在hadoop安装过之后，又再一次格式化，但是

datanode的data目录没有清空）
		3、datanode启动失败之二：用root身份启动过hadoop并且上传了数据，导致dfs文件夹的访问权限出现异常
```
##4、开发hdfs的api操作客户端
导入hadoop的核心包和依赖包以及common的核心包和依赖包
```
package com.baidu.hadoop;

public class HDFSUtils {

  //获得FileSystem实例
  FileSystem fs= null;
  
  @Before
  public void init() throws Exception{
    Configuration conf = new Configuration();
    //set replication num
    conf.set("dfs.replication","1");
    //set catelog of root
    conf.set("fs.defaultFS","hdfs://gaozhe:9000/");
    
    fs= FileSystem.get(conf);
  }
  /**
   * 文件上传
   * @throws Exception
   */
  @Test
  public void fileUpload() throws Exception{
    
    Path f= new Path("/baby.png");
    FSDataOutputStream os= fs.create(f);
    
    FileInputStream is= new FileInputStream(new File("/home/hadoop/image/129.png"));
    IOUtils.copy(is, os);
    
    //简单写法
    Path src= new Path("/home/hadoop/image/129.png");
    Path dst= new Path("/baby1.png");
    fs.copyFromLocalFile(src, dst);
  }
  
  /**
   * 文件下载
   * @throws Exception
   */
  @Test
  public void download() throws Exception{
    FSDataInputStream inputStream = fs.open(new Path("/baby.png"));
    FileOutputStream fileOutputStream= new FileOutputStream("/home/hadoop/image/baby.png");
    IOUtils.copy(inputStream, fileOutputStream);
    
    //简单写法
//    fs.copyToLocalFile(src, dst);
  }
  /**
   * 创建文件夹
   * @throws Exception
   */
  @Test
  public void mkDir() throws Exception{
    fs.mkdirs(new Path("/home/hadoop/test/test01"));
  }
  
  /**
   * 删除文件或文件夹
   * @throws Exception
   */
  @Test
  public void rmFileOrDir() throws Exception{
    fs.delete(new Path("/home"),true);//wether or not digui delete
  }
  
  /**
   * 查看文件
   * @throws Exception
   * @throws IllegalArgumentException 
   * @throws FileNotFoundException 
   */
  @Test
  public void listFiles() throws Exception{
    RemoteIterator<LocatedFileStatus> listFiles = fs.listFiles(new Path("/"), true);
    while(listFiles.hasNext()){
      LocatedFileStatus locatedFileStatu = listFiles.next();
      System.out.print(locatedFileStatu.getPath());
      
      BlockLocation[] locations = locatedFileStatu.getBlockLocations();
      for(int i=0; i< locations.length; i++){
        BlockLocation location = locations[i];
        String[] hosts = location.getHosts();
        
        for(int j=0; j< hosts.length; j++){
          System.out.println(hosts[j]);
        }
      }
    }
  }
  public static void main(String[] args) throws Exception {
    Configuration conf = new Configuration();
    conf.set("fs.defaultFS", "hdfs://192.168.2.6:9000/");
    FileSystem fs = FileSystem.get(conf);

    FSDataInputStream is = fs.open(new Path("/eclipse-jee-luna-SR1-linux-gtk.tar.gz"));

    FileOutputStream os = new FileOutputStream(new File("c:/eclipse.tgz"));
    
    IOUtils.copy(is, os);

  }
}
```
##5、RPC机制
###RPC（Remote Procedure Call Protocol）——远程过程调用协议
```
A  它是一种通过网络从远程计算机程序上请求服务，而不需要了解底层网络技术的协议。RPC协议假定某些传输协议的存在，如TCP或UDP，
为通信程序之间携带信息数据。在OSI网络通信模型中，RPC跨越了传输层和应用层。RPC使得开发包括网络分布式多程序在内的应用程序更加容易。

B  RPC采用客户机/服务器模式。请求程序就是一个客户机，而服务提供程序就是一个服务器。首先，客户机调用进程发送一个有进程参数的调用信息到服务进程，
然后等待应答信息。在服务器端，进程保持睡眠状态直到调用信息的到达为止。当一个调用信息到达，服务器获得进程参数，计算结果，发送答复信息，
然后等待下一个调用信息，最后，客户端调用进程接收答复信息，获得进程结果，然后调用执行继续进行。

C  hadoop的整个体系结构就是构建在RPC之上的(见org.apache.hadoop.ipc)。
```
###HDFS读过程
[![](../html/image/读过程.png)]
```
初始化FileSystem，然后客户端(client)用FileSystem的open()函数打开文件
FileSystem用RPC调用元数据节点，得到文件的数据块信息，对于每一个数据块，元数据节点返回保存数据块的数据节点的地址。
FileSystem返回FSDataInputStream给客户端，用来读取数据，客户端调用stream的read()函数开始读取数据。
DFSInputStream连接保存此文件第一个数据块的最近的数据节点，data从数据节点读到客户端(client)
当此数据块读取完毕时，DFSInputStream关闭和此数据节点的连接，然后连接此文件下一个数据块的最近的数据节点。
当客户端读取完毕数据的时候，调用FSDataInputStream的close函数。
在读取数据的过程中，如果客户端在与数据节点通信出现错误，则尝试连接包含此数据块的下一个数据节点。
失败的数据节点将被记录，以后不再连接。
```
###HDFS写过程
[![](../html/image/写过程.png)]
```
初始化FileSystem，客户端调用create()来创建文件
FileSystem用RPC调用元数据节点，在文件系统的命名空间中创建一个新的文件，元数据节点首先确定文件原来不存在，并且客户端有创建文件的权限，然后创建新文件。
FileSystem返回DFSOutputStream，客户端用于写数据，客户端开始写入数据。
DFSOutputStream将数据分成块，写入data queue。data queue由Data Streamer读取，并通知元数据节点分配数据节点，用来存储数据块(每块默认复制3块)。
分配的数据节点放在一个pipeline里。Data Streamer将数据块写入pipeline中的第一个数据节点。第一个数据节点将数据块发送给第二个数据节点。
第二个数据节点将数据发送给第三个数据节点。
DFSOutputStream为发出去的数据块保存了ack queue，等待pipeline中的数据节点告知数据已经写入成功。
当客户端结束写入数据，则调用stream的close函数。此操作将所有的数据块写入pipeline中的数据节点，并等待ack queue返回成功。最后通知元数据节点写入完毕。
如果数据节点在写入的过程中失败，关闭pipeline，将ack queue中的数据块放入data queue的开始，当前的数据块在已经写入的数据节点中被元数据节点赋予新的标示，
则错误节点重启后能够察觉其数据块是过时的，会被删除。失败的数据节点从pipeline中移除，另外的数据块则写入pipeline中的另外两个数据节点。
元数据节点则被通知此数据块是复制块数不足，将来会再创建第三份备份。

```
###案例
[![](../html/image/RPC实现机制.png)]
####服务端
接口
```
package com.baidu.rpc;

public interface LoginService {
  //要事先设置好版本号
  public static final long versionID=1L;
  public String login(String username ,String password);
}

```
实现类
```
package com.baidu.rpc;

public class LoginServiceImpl implements LoginService{

  @Override
  public String login(String username, String password) {
    
    return username+"login successfully";
  }
}

```
发布服务
```
package com.baidu.server;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.ipc.RPC;
import org.apache.hadoop.ipc.RPC.Builder;
import org.apache.hadoop.ipc.RPC.Server;

public class RPCServer {
  
  public static void main(String[] args) throws Exception{
    
    //用RPC框架拿到一个server的builder（server构造器）
    Builder builder = new RPC.Builder(new Configuration());

    /*给构造器传递一些必要的构造参数,服务端所监听的 ip地址和端口号，服务的真实业务实例，业务接口 */
    builder.setBindAddress("yun-11").setPort(10000).setInstance(new LoginServiceImpl()).setProtocol(LoginServiceInterface.class);
    
    //再用builder创建服务实例
    Server server = builder.build();
    
    //启动该服务实例
    server.start();
    
    //查看服务发布的端口号 netstat -natp
  }
}

```
####客户端
服务端接口（协议）注：包名必须和服务端的包名一致
```
package com.baidu.rpc;

public interface LoginService {
  //设置版本号，必须和服务端的版本号一致
  public static final long versionID=1L;
  public String login(String username ,String password);
}
```
调用服务端的方法
```
package com.baidu.rpc;

public class LoginController {

  public static void main(String[] args) throws Exception {
    
    //通过RPC的工具拿到服务的一个动态代理对象（是本端的socket程序的动态代理，但是它也实现了服务的接口协议）
    LoginService loginService = 
        RPC.getProxy(LoginService.class, 1L
            ,new InetSocketAddress("192.168.2.6", 10000), new Configuration());
    //用这个动态代理对象调用服务方法
    String result = loginService.login("angelababy", "123456");
    
    System.out.println(result);
    
    //最好关闭连接
    RPC.stopProxy(loginService);
  }
}

```
##6、跟踪hdfs客户端操作的一些源码
```
public static void main(String[] args) throws Exception {
    Configuration conf = new Configuration();
    conf.set("fs.defaultFS", "hdfs://192.168.2.6:9000/");
    FileSystem fs = FileSystem.get(conf);

    FSDataInputStream is = fs.open(new Path("/eclipse-jee-luna-SR1-linux-gtk.tar.gz"));

    FileOutputStream os = new FileOutputStream(new File("c:/eclipse.tgz"));
    
    IOUtils.copy(is, os);
```
FileSystem.get(conf);的源码分析
[![](../html/image/getfs流程.png)]
FileSystem.get(conf);流程
[![](../html/image/fsget流程.png)]
hdfs下载数据源码流程
[![](../html/image/hdfs下载数据源码流程.png)]


