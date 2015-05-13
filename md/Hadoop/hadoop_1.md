[笔记列表][anchor-id]

[anchor-id]: file:///D:/1125/%E7%AC%94%E8%AE%B0/note/html/
[anchor-cur]: #
[TOC]

## [ hadoop第一天笔记][anchor-cur]

+ 在windows的hosts中添加映射
    + `192.168.88.4` hadoop 
+ 修改linux主机名后,快速生效
    + `sudo hostname hadoop-1` 
+ 关闭CentOS图形化界面
    + `$ sudo vi /etc/inittab`
        + 修改成 `id:3:initdefault:`


Yarn 资源调度
+ ResourceManger
+ NodeManager

hadoop jar包位置
+ 核心 `hadoop-2.4.1\share\hadoop\hdfs\hadoop-hdfs-2.4.1.jar`
+ 依赖 `hadoop-2.4.1\share\hadoop\hdfs\lib`下所有
+ 公共组件 `hadoop-2.4.1\share\hadoop\common\hadoop-common-2.4.1.jar`
+ 依赖 `hadoop-2.4.1\share\hadoop\common\lib`下所有
cn.dishui.hadoop.wc.WordCountRunner

hadoop 运行jar包

hadoop jar wc.jar cn.dishui.hadoop.wc.WordCountRunner