[TOC]
##第一章：关机/重启命令
```
    shutdown系统关机 
    -r 关机后立即重启
    -h 关机后不重新启动
    halt 关机后关闭电源 
    reboot 重新启动
```
##第二章：常用命令
```
命令格式：命令  -选项  参数
如：ls  -la  /usr
ls：显示文件和目录列表(list)
常用参数：
-l      (long)
-a  (all)         注意隐藏文件、特殊目录.和..   
-t      (time)

top 显示当前系统中耗费资源最多的进程 
ps 显示瞬间的进程状态
-e /-A 显示所有进程，环境变量
-f 全格式
-a 显示所有用户的所有进程（包括其它用户）
-u 按用户名和启动时间的顺序来显示进程
-x 显示无控制终端的进程
kill 杀死一个进程
kill -9 pid
df 显示文件系统磁盘空间的使用情况
du 显示指定的文件（目录）已使用的磁盘空间的总
-h文件大小以K，M，G为单位显示（human-readable）
-s只显示各档案大小的总合（summarize）
free 显示当前内存和交换空间的使用情况 
netstat 显示网络状态信息
-a 显示所有连接和监听端口
-t (tcp)仅显示tcp相关选项
-u (udp)仅显示udp相关选项
-n 拒绝显示别名，能显示数字的全部转化成数字。
-p 显示建立相关链接的程序名
ifconfig 网卡网络配置详解 
ping 测试网络的连通性 
```
##第三章：Linux命令的分类
```
    内部命令：属于Shell解析器的一部分
        cd 切换目录（change directory）
        pwd 显示当前工作目录（print working directory）
        help 帮助
    外部命令：独立于Shell解析器之外的文件程序
        ls 显示文件和目录列表（list）
        mkdir 创建目录（make directoriy）
        cp 复制文件或目录（copy）
    查看帮助文档
        内部命令：help + 命令（help cd）
        外部命令：man + 命令（man ls）
 ```       
##第四章：备份压缩命令
```
    gzip 压缩（解压）文件或目录，压缩文件后缀为gz 
    bzip2 压缩（解压）文件或目录，压缩文件后缀为bz2 
    tar 文件、目录打（解）包
    
gzip命令：
    命令格式：gzip [选项] 压缩（解压缩）的文件名
    -d将压缩文件解压（decompress）
    -l显示压缩文件的大小，未压缩文件的大小，压缩比（list）
    -v显示文件名和压缩比（verbose）
    -num用指定的数字num调整压缩的速度，-1或--fast表示最快压缩方法（低压缩比），-9或--best表示最慢压缩方法（高压缩比）。系统缺省值为6
    
bzip2命令：
    命令格式：bzip2 [-cdz] 文档名
    -c将压缩的过程产生的数据输出到屏幕上
    -d解压缩的参数（decompress）
    -z压缩的参数（compress）
    -num 用指定的数字num调整压缩的速度，-1或--fast表示最快压缩方法（低压缩比），-9或--best表示最慢压缩方法（高压缩比）。系统缺省值为6
    
tar命令：
    -c 建立一个压缩文件的参数指令（create）
    -x 解开一个压缩文件的参数指令（extract）
    -z 是否需要用 gzip 压缩
    -j 是否需要用 bzip2 压缩
    -v 压缩的过程中显示文件（verbose）
    -f 使用档名，在 f 之后要立即接档名（file）
    
例子：
    压缩
       gzip -cvf *
       bzip2 -cvf *
    解压
       gunzip -xvf *.gz   /gzip -d -xvf *.gz
       bunzip2 -xvf *.bz2 /bzip2 -d -xvf *.bz2
    打包
       tar -cvf *.tar *
    解包
       tar -xvf *.tar
    打包并压缩
       tar -zcvf *.tar.gz *   /tar -jcvf *.tar.gz2 *
    解压并解包 
       tar -zxvf *.tar.gz     /tar -jxvf *.tar.bz2 -C 目录
```
##第五章：VIM文本编辑器
```
    vi / vim是Unix / Linux上最常用的文本编辑器而且功能非常强大。
    只有命令，没有菜单。
```
[![](../pic/1.png)]
[![](../pic/2.png)]
[![](../pic/3.png)]
[![](../pic/4.png)]
[![](../pic/5.png)]
[![](../pic/6.png)]

##第六章：用户和组账户管理
```
    linux操作系统是一个多用户操作系统，它允许多用户同时登录到系统上并使用资源。系统会根据账户来区分每个用户的文件，进程，任务和工作环境，使得每个用户工作都不受干扰。
    配置文件：
    保存用户信息的文件：/etc/passwd
    保存密码的文件：/etc/shadow
    保存用户组的文件：/etc/group
    保存用户组密码的文件：/etc/gshadow
    用户配置文件：/etc/default/useradd
```
[![](../pic/7.png)]
##第七章：Linux用户分类
```
超级用户：（root，UID=0）
普通用户：（UID在500到60000）
伪用户：（UID在1到499）
系统和服务相关的：bin、daemon、shutdown等
进程相关的：mail、news、games等
为用户不能登陆系统，而且没有宿主目录
```
[![](../pic/8.png)]
```
用户组：
每个用户至少属于一个用户组
每个用户组可以包含多个用户
同一个用户组的用户享有该组共有的权限

```
[![](../pic/9.png)]
###操作用户命令
```
添加用户命令：useradd
-u 指定组ID（uid）
-g 指定所属的组名（gid）
-G 指定多个组，用逗号“，”分开（Groups）
-c 用户描述（comment）
-e 失效时间（expire date）
例子：
useradd -u 888 -g users -G sys,root -c "hr zhang" zhangsan
passwd zhangsan

修改用户命令：usermod（user modify）
-l 修改用户名 （login）usermod -l a b（b改为a）
-g 添加组 usermod -g sys tom
-G添加多个组 usermod -G sys,root tom
–L 锁定用户账号密码（Lock）
–U 解锁用户账号（Unlock）
删除用户命令：userdel（user delete）
-r 删除账号时同时删除目录（remove）

添加组：groupadd
-g 指定gid
修改组：groupmod
-n 更改组名（new group）
删除组：groupdel
groups 显示用户所属组
```
##第八章：权限管理
```
三种基本权限 
r 读权限（read）
w 写权限（write）
x 执行权限 （execute）
chmod修改文件权限命令（change mode）
    参数：-R 下面的文件和子目录做相同权限操作（Recursive递归的）
    例如：chmod  u+x  a.txt
用数字来表示权限（r=4，w=2，x=1，-=0）
    例如：chmod  750  b.txt
    rwx用二进制表示是111，十进制4+2+1=7
    r-x用二进制表示是101，十进制4+0+1=5
```
[![](../pic/10.png)]
##第九章： RPM软件包管理
```
RPM是RedHat Package Manager（RedHat软件包管理工具）的缩写，这一文件格式名称虽然打上了RedHat的标志，但是其原始设计理念是开放式的，现在包括RedHat、CentOS、SUSE等Linux的分发版本都有采用，可以算是公认的行业标准了。RPM文件在Linux系统中的安装最为简便
```
###RPM命令使用
```
 rpm的常用参数
i：安装应用程序（install）
e：卸载应用程序（erase）
vh：显示安装进度；（verbose   hash） 
U：升级软件包；（update） 
qa: 显示所有已安装软件包（query all）
结合grep命令使用
例子：rpm  -ivh  gcc-c++-4.4.7-3.el6.x86_64.rpm
```
###Linux 网络配置
```
DEVICE="eth0"
BOOTPROTO=“static"
HWADDR="00:0C:29:62:4C:2B"
IPV6INIT="yes"
NM_CONTROLLED="yes"
ONBOOT="yes"
TYPE="Ethernet"
UUID="1acc3359-b1fd-4ac8-b044-58b5fe5a16ce“
IPADDR="192.168.24.20"
NETMASK="255.255.255.0"
GATEWAY="192.168.24.1"
DNS1="8.8.8.8"
DNS2="8.8.4.4"
```
###YUM命令
```
Yum（全称为 Yellow dog Updater, Modified）是一个在Fedora和RedHat以及SUSE、CentOS中的Shell前端软件包管理器。基於RPM包管理，能够从指定的服务器自动下载RPM包并且安装，可以自动处理依赖性关系，并且一次安装所有依赖的软件包，无须繁琐地一次次下载、安装。
例子（需要上网，没有网络可以建本地源）：
yum  install  gcc-c++
yum  remove  gcc-c++
yum  update  gcc-c++

```