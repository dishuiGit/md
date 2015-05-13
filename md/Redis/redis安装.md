[TOC]

## Redis安装
1. 到官网下载最新stable版
2. 解压源码并进入目录      
    + `tar -zxvf redis-2.8.19.tar.gz  -C  ./redis-src/`
3. `make`
4.  <font color='green'>可选</font>  make test （可能出现need tcl>8.4,yum install tcl）
5. 安装到指定目录,如 `/usr/local/redis`
    make PREFIX=/usr/local/redis install

6. 拷贝一份配置文件到安装目录下
+ 切换到源码目录，里面有一份配置文件 redis.conf
+ 然后将其拷贝到安装路径下
    + `cp redis.conf /usr/local/redis/`




安装完后，在/usr/local/redis/bin下有几个可执行文件
redis-benchmark  ----性能测试工具
redis-check-aof  ----检查修复aof文件 appendonly file
redis-check-dump ----检查快照持久化文件
redis-cli  ----命令行客户端
redis-server   ----redis服务器启动命令




运行
bin/redis-server
(默认前端运行模式。后台进程模式： 修改conf   daemonize yes)
也可以手动控制后端运行，按这种形式来启动：   nohup bin/redis-server 1 > /dev/null 2 > &1 &


连接
bin/redis-cli
