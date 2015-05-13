[笔记列表][anchor-id]

[anchor-id]: file:///D:/1125/%E7%AC%94%E8%AE%B0/note/html/
[anchor-cur]: #
[TOC]


### [ SqlMapConfig.xml][anchor-cur]

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
"http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>


    <!-- 全局设置 -->
    <settings>
        <!-- 延迟加载总开关 -->
        <setting name="lazyLoadingEnabled" value="true" />  
        <!-- 设置按需加载 -->
        <setting name="aggressiveLazyLoading" value="true"/>
    </settings>
    <!-- 配置别名 -->
    <typeAliases>
<!--        <typeAlias type="cn.itcast.mybatis.po.User" alias="User"/> -->
<!--        <typeAlias type="cn.itcast.mybatis.po.QueryVo" alias="QueryVo"/>
大小第一个字母 QueryVo  小写第一个字母  queryVo
 -->
        <package name="cn.itcast.mybatis.po"/>
    </typeAliases>
    
    <!-- 和spring整合后 environments配置将废除-->
    <environments default="development">
        <environment id="development">
        <!-- 使用jdbc事务管理-->
            <transactionManager type="JDBC" />
        <!-- 数据库连接池-->
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jdbc.Driver" />
                <property name="url" value="jdbc:mysql://localhost:3306/mybatis?characterEncoding=utf-8" />
                <property name="username" value="root" />
                <property name="password" value="root" />
            </dataSource>
        </environment>
    </environments>
    
    <!-- Mapper的位置    mapper  classpath 下 
    <mappers>
        <mapper resource="sqlmap/User.xml"/>
    </mappers>
     -->
     <mappers>
<!--        <mapper resource="cn/itcast/mybatis/dao/UserDao.xml" /> -->
        <package name="cn/itcast/mybatis/dao"/>
     </mappers>
    
</configuration>
```



### [ User.xml][anchor-cur]

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
"http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!-- 工作空间 namespace  对Sql语句隔离  学习Mapper动态代理方式开发时候  大作用  -->
<mapper namespace="cn.itcast.mybatis.dao.UserDao">

    <!-- 自定义映射关系 -->
    <resultMap type="User" id="useridResultMap">
        <!-- 指定ID映射
        column : 表示数据库字段
        property : 表示POJO属性字段
         -->
        <id column="id" property="id"/>
        <result column="usernaem_" property="username"/>
        <result column="birthday" property="birthday"/>
        <result column="sex" property="sex"/>
        <result column="address" property="address"/>
    </resultMap>

    <!-- 一个Select标签就一个 Statement  int Integer  -->
    <select id="findUserById" parameterType="int" resultMap="useridResultMap">
        select id,username usernaem_,birthday,sex,address from user where id = #{w}
    </select>
    <!-- 通过用户名模糊查询用户对象  返回用户多个 -->
    <select id="findUserByUserName" parameterType="String" resultType="cn.itcast.mybatis.po.User">
        select * from user where username like  "%"#{v}"%"
    </select>
    <!-- 一个Select标签就一个 Statement  int Integer
        包装类: ognl 表达式
      -->
    <select id="findUserByQueryVoId" parameterType="QueryVo" resultType="User">
        select * from user where id = #{user.id}
    </select>
    <!-- 一个Select标签就一个 Statement  int Integer   Map  map  = new HashMap()
        包装类: ognl 表达式
      -->
    <select id="findUserByHashMapId" parameterType="hashmap" resultType="User">
        select * from user where id = #{id}
    </select>
    
    <!-- 动态Sql  jdbc 问题  变化太大 
        private int id;
        private String username;// 用户姓名
        private String sex;// 性别
        private Date birthday;// 生日
        private String address;// 地址
        
        <where>  : 可以帮助我们去掉第一个And
    
    -->
    <select id="findUserByUser" parameterType="User" resultType="User">
        select
        * 
        from user where id in (1,10,11,15,17)
        <where>
            <if test="username != null and username != ''">
                username = #{username}  
            </if>
            <if test="id != null and id != ''">
                and id = #{id}
            </if>
        </where>
        
    </select>
    
    <!-- Foreach 标签  遍历 
    collection:集合
    open:遍历之前东西
    close: 遍历之前东西
    separator : 隔点
    item : 集合中的每一项
    (1,10,11,15,17)
    -->
    <select id="findUserByIds" parameterType="Integer" resultType="User">
        select
        * 
        from user 
        where 
        id in 
        <foreach collection="list" open="(" close=")" separator="," item="id">
            #{id}
        </foreach>
    
    </select>
    
    <!-- 
        private User user;
    
        List<Integer> ids;
    
     -->
    
    <select id="findUserByQueryIds" parameterType="QueryVo" resultType="User">
        select
        * 
        from user 
        where 
        id in 
        <foreach collection="ids" open="(" close=")" separator="," item="id">
            #{id}
        </foreach>
    
    </select>
    <!-- 数组 -->
    <select id="findUserByArrayIds" parameterType="Object" resultType="User">
        select
        * 
        from user 
        <include refid="query_where"/>
    </select>
    <!-- sql片段 -->
    <sql id="query_where">
        where 
        id in 
        <foreach collection="array" open="(" close=")" separator="," item="id">
            #{id}
        </foreach>
    </sql>
    
    <!-- 一对一映射 
    <resultMap type="OrdersCustom" id="orderUserResultMap">
        <id column="id" property="id"/>
        <result column="user_id" property="userId"/>
        <result column="number" property="number"/>
        <result column="createtime" property="createtime"/>
        <result column="username" property="username"/>
        <result column="address" property="address"/>
    </resultMap>
    -->
    <resultMap type="Orders" id="orderUserResultMap">
        <id column="id" property="id"/>
        <result column="user_id" property="userId"/>
        <result column="number" property="number"/>
        <result column="createtime" property="createtime"/>
        <!-- 用户名  及 地址
        association : 表示当前对象 有一个 用户对象   一对一关系
         -->
        <association property="user" javaType="User">
            <result column="username" property="username"/>
            <result column="address" property="address"/>
        </association>
    </resultMap>
    <!-- 一对一关联查询  映射POJO -->
    <select id="findOrderUser" resultMap="orderUserResultMap">
        select orders.*,user.username,user.address
        from user,orders
        where user.id = orders.user_id
    </select>
    
    <!-- 一对多 -->
    <resultMap type="User" id="userOrderResultMap">
        <id column="id" property="id"/>
        <result column="username" property="username"/>
        <result column="address" property="address"/>
        <!-- List集合  映射 
        collection : 表示映射 集合类
        property :表示 集合类的名称
        ofType : 表示集合类泛型的类型
        -->
        <collection property="ordersList" ofType="Orders">
            <id column="orderId" property="id"/>
            <result column="number" property="number"/>
            <result column="createtime" property="createtime"/>
        </collection>
    </resultMap>
    <!-- 一对多  关联查询  映射POJO  以用户对象为中心  相对于  订单   -->
    <select id="findUserOrders" resultMap="userOrderResultMap">
        select user.*,orders.id orderId,orders.number,orders.createtime
        from user,orders
        where user.id = orders.user_id
    </select>
    
    <!-- 多对多   -->
    
    <resultMap type="User" id="useridResultMapExtends">
        <!-- 指定ID映射
        column : 表示数据库字段
        property : 表示POJO属性字段
         -->
        <id column="id" property="id"/>
        <result column="username" property="username"/>
        <result column="birthday" property="birthday"/>
        <result column="sex" property="sex"/>
        <result column="address" property="address"/>
    </resultMap>
    <resultMap type="User" id="userOrderDetailItemsResultMap" extends="useridResultMapExtends">
        <!-- 订单映射 -->
        <collection property="ordersList" ofType="Orders">
            <id column="ordersId" property="id"/>
            <result column="number" property="number"/>
            <result column="createtime" property="createtime"/>
            <!-- 与订单详情一对多关系  -->
            <collection property="orderDetailList" ofType="OrderDetail">
                <id column="orderdetailId" property="id"/>
                <result column="items_num" property="itemsNum"/>
                <!-- 与商品信息  一对一关系 -->
                <association property="items" javaType="Items">
                    <id column="itemsId" property="id"/>
                    <result column="name" property="name"/>
                    <result column="price" property="price"/>
                    <result column="detail" property="detail"/>
                </association>
            </collection>
        </collection>
    </resultMap>
    
    <!-- 多对多  关联查询   以用户对象为中心   -->
    <select id="findUserOrdersDetailItems" resultMap="userOrderDetailItemsResultMap">
            select user.*,
            orders.id ordersId ,orders.number,orders.createtime,
            orderdetail.id orderdetailId,orderdetail.items_num,
            items.id itemsId,items.name,items.price,items.detail
            from user,orders,orderdetail,items
            
            where user.id = orders.user_id 
            and orders.id = orderdetail.orders_id
            and orderdetail.items_id = items.id
    </select>
    
    <!-- 延迟加载映射 -->
    <resultMap type="Orders" id="orderUserMap">
            <id column="id" property="id"/>
            <!-- 用户ID -->
            <result column="user_id" property="userId"/>
            <result column="number" property="number"/>
            <result column="createtime" property="createtime"/>
            <!-- 一对一 -->
            <association property="user" javaType="User" select="findOrderAndUserById" column="user_id">
                <id column="id" property="id"/>
                <result column="username" property="username"/>
                <result column="birthday" property="birthday"/>
                <result column="sex" property="sex"/>
                <result column="address" property="address"/>
            </association>
    </resultMap>
    
    <!-- 以订单为中心  延迟加载用户信息 -->
    <select id="findOrder" resultMap="orderUserMap">
        select * from orders
    </select>
    <!-- 通过ID  查询用户信息 -->
    <select id="findOrderAndUserById" parameterType="int" resultMap="useridResultMapExtends">
        select id,username,birthday,sex,address from user where id = #{w}
    </select>
    
</mapper>



```