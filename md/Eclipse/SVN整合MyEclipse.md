## SVN整合Eclipse/MyEclipse

 1. 通过官网subclipse.tigris.org下载插件文件
`eclipse_svn_site-1.6.5.zip`		`myeclipse-svn-site-1.6.16.zip`

 2. 解压压缩包中features与plugins文件夹，并复制到任意目录X。注意目录中不能包含有中文或空格字符。

 3. 在MyEclipse安装目录的dropins目录下，创建文本文件，文件名任意，扩展名为.link，录入svn.link，并编辑内容如下：
>	+ `path=X`   注意：路径中的分隔符使用\\\
>	+ `path=E:\\MyEclipse\\myPlugin\\svn`

 4. 删除MyEclipse安装目录下的`configuration\org.eclipse.update`目录，重新加载配置信息

 5. 启动MyEclipse，视图中添加了SVN的管理视图模式
