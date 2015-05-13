[笔记列表][anchor-id]

[anchor-id]: file:///D:/1125/%E7%AC%94%E8%AE%B0/note/html/
[anchor-cur]: #
[TOC]

## [ 准备Linux环境][anchor-cur]

#### 1.0 VMware网络配置
+ 点击VMware的edit菜单，
+ 然后点击 `virtual network editor`
	* 确认子网网段  `subnet ip`  为 `192.168.2.0`
	* 选择 `NAT setting`，进去之后 确认`gateway ip` 为 `192.168.2.1`
	* 修改windows的ip 
		* 修改`vmnet8`这块网卡的地址为    __ip__：`192.168.2.2`   __网关__:`192.168.2.1` 
		
		* __子网掩码:__ `255.255.255.0`  __dns__：`192.168.2.1`

----------------------------------------	
	
#### 1.1 修改IP
两种方式：
<span class='d_red'>第一种</span>：通过Linux图形界面进行修改（强烈推荐）
- 第一种:进入Linux图形界面  
	+ 右键点击右上方的两个小电脑  
	+ 点击Edit connections  
	+ 选中当前网络System eth0  
		* 点击edit按钮
			* 选择IPv4 
			* method选择为manual  
				* 点击add按钮  
				* 添加IP：`192.168.88.4 `
				* 子网掩码：`255.255.255.0 `
				* 网关：192.168.88.1  
				* apply
	+ 重启`linux`系统  或者 （`su` 切换到`root`   然后敲命令：service network restart  ）



- 第二种:修改配置文件方式（屌丝程序猿专用）

		sudo vi /etc/sysconfig/network-scripts/ifcfg-eth0

		DEVICE="eth0"
		BOOTPROTO="static"               ###
		HWADDR="00: 0C :29 :3C :BF :E7"
		IPV6INIT="yes"
		NM_CONTROLLED="yes"
		ONBOOT="yes"
		TYPE="Ethernet"
		UUID="ce22eeca-ecde-4536-8cc2-ef0dc36d4a8c"
		IPADDR="192.168.2.11"           ###
		NETMASK="255.255.255.0"          ###
		GATEWAY="192.168.2.1"            ###
		

			 
#### 1.2修改Linux服务器主机名
 注：首先要将普通用户`hadoop`加入到`/etc/sudoers`文件中去
- `su`切换到`root`用户
	+ vi `/etc/sudoers`  在`root`行下添加一行
		`root    ALL=(ALL)       ALL`
		`hadoop  ALL=(ALL)       ALL`
	+ 然后保存
	+ 然后退出`root`用户，命令是  `exit`

- 然后就可以用普通用户使用`sudo`命令做系统配置了 	
		sudo vi /etc/sysconfig/network
		
		NETWORKING=yes
		HOSTNAME=yun-11    ###

#### 1.3修改主机名和IP的映射关系

	sudo vi /etc/hosts
	添加以下行:	
	192.168.1.11	yun-11


#### 1.4关闭防火墙

- 查看防火墙状态
`sudo service iptables status`
- 关闭防火墙
`sudo service iptables stop`
- 查看防火墙开机启动状态
`sudo chkconfig iptables --list`
- 关闭防火墙开机启动
`sudo chkconfig iptables off`

#### 1.5重启Linux
	reboot

## [ 2.安装JDK][anchor-cur]

#### 2.1直接在CRT终端上传
- 按 `alt+p` 后出现`sftp`窗口
- 然后`put d:\xxx\yy\ll\jdk-7u_65-i585.tar.gz`
- 文件就会被传到服务器的当前用户主目录下

#### 2.2解压jdk
- 创建文件夹
	mkdir /home/hadoop/app
- 解压
	tar -zxvf jdk-7u55-linux-i586.tar.gz -C /home/hadoop/app
	
#### 2.3将java添加到环境变量中

	sudo vi /etc/profile
	#在文件最后添加
	export JAVA_HOME=/home/hadoop/app/jdk-7u_65-i585
	export PATH=$PATH:$JAVA_HOME/bin

	#刷新配置
	source /etc/profile
	
## 3.安装hadoop2.4.1

- 先上传`hadoop`的安装包到服务器上去`/home/hadoop/`
	+ 注意：hadoop2.x的配置文件`$HADOOP_HOME/etc/hadoop`

- 伪分布式需要修改5个配置文件

#### <span class='d_red'>3.1配置hadoop</span>

第一个：<span class='d_red'>`hadoop-env.sh`</span>
		
		vim hadoop-env.sh
		#第27行
		export JAVA_HOME=/home/hadoop/app/jdk1.7.0_65
	
第二个：<span class='d_red'>`core-site.xml`</span>
```xml
<!-- 指定HADOOP所使用的文件系统schema（URI），HDFS的老大（NameNode）的地址 -->
<property>
	<name>fs.defaultFS</name>
	<value>hdfs://192.168.88.4:9000</value> 
</property>
<!-- 指定hadoop运行时产生文件的存储目录 -->
<property>
	<name>hadoop.tmp.dir</name>
	<value>/home/hadoop/hadoop-2.4.1/tmp</value>
</property>
```		
第三个：<span class='d_red'>`hdfs-site.xml`</span>   <span class='d_red'>`hdfs-default.xml`</span>  (3)
```xml
<!-- 指定HDFS副本的数量 -->
<property>
	<name>dfs.replication</name>
	<value>1</value>
</property>
```		
第四个：<span class='d_red'>`mapred-site.xml`</span>
	`mv mapred-site.xml.template mapred-site.xml`
	`vim mapred-site.xml`
```xml
<!-- 指定mr运行在yarn上 -->
<property>
	<name>mapreduce.framework.name</name>
	<value>yarn</value>
</property>
```		
第五个：<span class='d_red'>`yarn-site.xml`</span>
```xml
<!-- 指定YARN的老大（ResourceManager）的地址 -->
<property>
	<name>yarn.resourcemanager.hostname</name>
	<value>yun-11</value>
</property>
	<!-- reducer获取数据的方式 -->
<property>
	<name>yarn.nodemanager.aux-services</name>
	<value>mapreduce_shuffle</value>
</property>
```    	
#### 3.2将hadoop添加到环境变量

	sudo vi /etc/proflie
		添加
		export HADOOP_HOME=/home/hadoop/hadoop-2.4.1
		export PATH=$PATH:$JAVA_HOME/bin:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
	然后刷新
	source /etc/profile

#### 3.3格式化namenode（是对namenode进行初始化）
		hdfs namenode -format (或者 hadoop namenode -format)
		怎样才算成功呢？ 要看到这句话：
		INFO common.Storage: Storage directory /home/hadoop/app/hadoop-2.4.1/temp/dfs/name has been successfully formatted.
	
#### 3.4启动hadoop
- 先启动HDFS
	+ `sbin/start-dfs.sh`

- 再启动YARN
	+ `sbin/start-yarn.sh`
	
#### 3.5验证是否启动成功
	使用jps命令验证
	27408 NameNode
	28218 Jps
	27643 SecondaryNameNode
	28066 NodeManager
	27803 ResourceManager
	27512 DataNode

	http://192.168.1.101:50070 （HDFS管理界面）
	http://192.168.1.101:8088 （MR管理界面）
	
## [ 4.配置ssh免登陆][anchor-cur]
- 生成ssh免登陆密钥
- 进入到我的`home`目录
`cd ~/.ssh`

- `ssh-keygen -t rsa` （四个回车）
	>执行完这个命令后，会生成两个文件`id_rsa`（私钥）、`id_rsa.pub`（公钥）
	>将公钥拷贝到要免登陆的机器上
	>`ssh-copy-id localhost`
	
