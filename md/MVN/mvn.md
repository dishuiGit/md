## MVN配置

+ 设置MAVEN_HOME环境变量
	+ 升级时只需要下载最新版本，解压缩后重新设置MAVEN_HOME环境变量即可

+ 设置MAVEN_OPTS环境变量
	+ MAV启动内存配置
	
		`-Xms128m -Xmx512m`

+ 配置用户范围的settings.xml

	`MAVEN_HOME/conf/settings.xml` 全局的

+ `~/.m2/settings.xml` 
	+ 默认仓库：当前用户路径`C:\Users\[UserName]\.m2`
	
+ localRepository：
	* 用户仓库，用于检索依赖包路径

------------------------------------------------------

#### 用户Maven依赖包路径层次目录

><font color="red">需要上传图片</font>
>
>+ `settings.xml`文件必须与`maven`安装路径下的内容保持一致
>
>+ 配置中设置路径指向仓库目录
>
>	```xml
>		<localRepository>D:\maven\repository</localRepository>
>	```
>
>*注意*：
>	&nbsp;&nbsp;&nbsp;&nbsp;<font color="red">	用户级别的仓库在全局配置中一旦设置，全局配置将不再生效，转用用户所设置的仓库，否则使用默认路径仓库</font>

---------------------------------------------

## Maven约定

> + **src/main/java** —— 存放项目的.java文件
> 
> + **src/main/resources** —— 存放项目资源文件，如spring, hibernate配置文件

> + **src/test/java** —— 存放所有测试.java文件，如JUnit测试类

> + **src/test/resources** —— 测试资源文件

> + **target** —— 项目输出位置

> + **pom.xml**——maven项目核心配置文件

---------------------------------------------

---------------------------------------------

---------------------------------------------

---------------------------------------------

---------------------------------------------
