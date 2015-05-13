[笔记列表][anchor-id]

[anchor-id]: file:///D:/1125/%E7%AC%94%E8%AE%B0/note/html/
[anchor-cur]: #
[TOC]

### [ 单独使用jdbc编程问题总结][anchor-cur]

```java
Public static void main(String[] args) {
    Connection connection = null;
    PreparedStatement preparedStatement = null;
    ResultSet resultSet = null;
    
    try {
        //加载数据库驱动
        Class.forName("com.mysql.jdbc.Driver");
        
        //通过驱动管理类获取数据库链接
        connection =  DriverManager.getConnection("jdbc:mysql://localhost:3306/mybatis?characterEncoding=utf-8", "root", "mysql");
        //定义sql语句 ?表示占位符
    String sql = "select * from user where username = ?";
        //获取预处理statement
        preparedStatement = connection.prepareStatement(sql);
        //设置参数，第一个参数为sql语句中参数的序号（从1开始），第二个参数为设置的参数值
        preparedStatement.setString(1, "王五");
        //向数据库发出sql执行查询，查询出结果集
        resultSet =  preparedStatement.executeQuery();
        //遍历查询结果集
        while(resultSet.next()){
            System.out.println(resultSet.getString("id")+"  "+resultSet.getString("username"));
        }
    } catch (Exception e) {
        e.printStackTrace();
    }finally{
        //释放资源
        if(resultSet!=null){
            try {
                resultSet.close();
            } catch (SQLException e) {
                // TODO Auto-generated catch block
                e.printStackTrace();
            }
        }
        if(preparedStatement!=null){
            try {
                preparedStatement.close();
            } catch (SQLException e) {
                // TODO Auto-generated catch block
                e.printStackTrace();
            }
        }
        if(connection!=null){
            try {
                connection.close();
            } catch (SQLException e) {
                // TODO Auto-generated catch block
                e.printStackTrace();
            }
        }

    }

}
```

#### jdbc编程步骤

1. 加载数据库驱动
1. 创建并获取数据库链接
1. 创建jdbc statement对象
1. 设置sql语句
1. 设置sql语句中的参数(使用preparedStatement)
1. 通过statement执行sql并获取结果
1. 对sql执行结果进行解析处理
1. 释放资源(resultSet、preparedstatement、connection)

#### jdbc<span class='d_red'>问题</span>总结如下

1. 数据库链接创建、释放频繁造成系统资源浪费从而影响系统性能，如果使用数据库链接池可解决此问题。
1. Sql语句在代码中硬编码，造成代码不易维护，实际应用sql变化的可能较大，sql变动需要改变java代码。
1. 使用preparedStatement向占有位符号传参数存在硬编码，因为sql语句的where条件不一定，可能多也可能少，修改sql还要修改代码，系统不易维护。
1. 对结果集解析存在硬编码（查询列名），sql变化导致解析代码变化，系统不易维护，如果能将数据库记录封装成pojo对象解析比较方便。

### [ Mybatis架构][anchor-cur]

![](../../pic/mybatis_01.PNG)

### [ Mybatis 流程][anchor-cur]
1. mybatis配置
    + <span class='d_red'>`SqlMapConfig.xml`</span> 
        ，此文件作为mybatis的全局配置文件，配置了mybatis的运行环境等信息。
        `mapper.xml`文件即sql映射文件，文件中配置了操作数据库的sql语句。
        此文件需要在`SqlMapConfig.xml`中加载。

1. 通过mybatis环境等配置信息构造<span class='d_green'>`SqlSessionFactory`</span>即会话工厂
1. 由会话工厂创建`sqlSession`即会话，操作数据库需要通过`sqlSession`进行。
1. mybatis底层自定义了`Executor`执行器接口操作数据库，`Executor`接口有两个实现，一个是基本执行器、一个是缓存执行器。
1. <span class='d_green'>`Mapped Statement`</span>也是mybatis一个底层封装对象，它包装了mybatis配置信息及`sql`映射信息等。`mapper.xml`文件中一个sql对应一个`Mapped Statement`对象，`sql`的`id`即是`Mapped statement`的`id`。
1. `Mapped Statement`对sql执行输入参数进行定义，包括`HashMap`、基本类型、`pojo`，`pojo`
    通过`pojo` `Statement`在执行`sql`前将输入的`java`对象映射至`sql`中，输入参数映射就是`jdbc`
    编程中对`preparedStatement`设置参数。
1. `Mapped Statement`对`sql`执行输出结果进行定义，包括`HashMap`、基本类型、`pojo`，`Executor`
    通过`Mapped` `Statement`在执行`sql`后将输出结果映射至`java`对象中，输出结果映射过程相当于`
    jdbc`编程中对结果的解析处理过程。

#### <span class='d_red'>`SqlMapConfig.xml`</span>配置文件

- ```xml
    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE configuration
    PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
    "http://mybatis.org/dtd/mybatis-3-config.dtd">
    <configuration>
        <!-- 和spring整合后 environments配置将废除-->
        <environments default="development">
            <environment id="development">
            <!-- 使用jdbc事务管理-->
                <transactionManager type="JDBC" />
            <!-- 数据库连接池-->
                <dataSource type="POOLED">
                    <property name="driver" value="com.mysql.jdbc.Driver" />
                    <property name="url" value="jdbc: mysql://localhost:3306/mybatis?characterEncoding=utf-8" />
                    <property name="username" value="root" />
                    <property name="password" value="123" />
                </dataSource>
            </environment>
        </environments>
        
    </configuration>
    ```