[TOC]

## 知识总结

1. Hibernate关联属性操作需要通过对象名称按照对象包含关系结构进行设定，例如上例中的em.dm.uuid即为em对象的dm属性的uuid属性进行赋值

2. Hibernate中的更新分为两种
	* 一种是物理更新
		- 物理更新指开发者提供需要更新的数据，不进行任何检测直接进行，部门模块所采用的更新即为物理更新，物理更新要求开发者指定更新过程中的所有数据，如果某个数据未进行指定，按null进行处理，这样容易引发数据库约束的错误或程序的报错（传递了null对象后，导致的错误操作）。物理更新也成为物理修改。
	* 一种是快照更新
		- 快照更新指开发者先根据待修改的数据的OID，进行查询，将原始数据加载在快照区，然后根据用户传递的数据，更新快照区中的数据，利用快照思想对原始数据进行修改。快照更新的优势在于即便用户为程序传递了某些错误的信息，如果此类数据不参与更新，则不会影响最终的数据，该类数据将被忽略，避免了物理更新模式中的错误现象的发生。快照更新也成为逻辑修改。


多对多:两张表中间加一张中间表

分页导入后;要创建隐藏域


程序没错,运行爆错,一定要检查配置文件 hbm.xml 

hibernate爆错:检查顺序 hbm.xml --> 程序

多对一

```xml
<many-to-one name="parentMM"  class="cn.itcast.erp.auth.menu.vo.MenuModel" column="parent_uuid"/>
```

多对多

```xml
        <!-- 角色 对 资源 [多]对多 -->
        <set name="ress" table="tbl_role_res">
            <key column="role_uuid" />
            <many-to-many class="cn.itcast.erp.auth.res.vo.ResModel" column="res_uuid" />
        </set>
```

>set的name与many-to-many的class对应
>`RoleModel`的`uuid`与`tbl_role_res`表的`role_uuid`连接
>many-to-many中的`ResModel`的uuid与`tbl_role_res`表的`res_uuid`连接

### 自关联查询

```xml
<hibernate-mapping>
    
    <class name="cn.itcast.erp.auth.menu.vo.MenuModel" table="tbl_menu">
        <id name="uuid">
            <generator class="native" />
        </id>
        <property name="name"/>
        <property name="url"/>
        <!-- 关系 [子菜单]到父菜单 多对一 -->
        <many-to-one name="parentMM" class="cn.itcast.erp.auth.menu.vo.MenuModel" column="parent_uuid"/>
        <!-- 关系 [父菜单]到子菜单 一对多 -->
        <!-- 每个菜单具有多个菜单项 -->
        <!-- 级联删除：断开关系，删除所有的子，删除父 -->
        <!-- 将级联删除设置为不对关联关系进行维护 -->
        <set name="mms">
            <key column="parent_uuid"/>
            <one-to-many class="cn.itcast.erp.auth.menu.vo.MenuModel" />
        </set>
        <!-- 关系 [菜单]到角色 多对多 -->
        <set name="roles" table="tbl_role_menu">
            <key column="menu_uuid"/>
            <many-to-many class="cn.itcast.erp.auth.role.vo.RoleModel" column="role_uuid" />
        </set>
        
    </class>
</hibernate-mapping>
```


### 级联删除

```java
    public void delete(MenuModel mm) {
        //当前mm对象中具有的数据有哪些？
        //uuid
        //parent,children对象是什么值？null
        //hibernate针对null对应的关系的处理策略（带有级联删除）
        
        //必须先将mm对象的关联关系找到，然后才能进行有效的级联删除
        mm = menuDao.get(mm.getUuid());
        menuDao.delete(mm);
    }

```

### 一对多

```xml
  <class name="cn.itcast.erp.invoice.supplier.vo.SupplierModel" table="tbl_supplier">
        <id name="uuid">
            <generator class="native" />
        </id>
        <property name="name"/>
        <property name="address"/>
        <property name="tele"/>
        <property name="contact"/>
        <property name="needs"/>
        <!--供应商对商品类别 [一]对多 -->
        <set name="gtms" table="tbl_goodstype">
            <key column="supplier_uuid" />
            <one-to-many class="cn.itcast.erp.invoice.goodstype.vo.GoodsTypeModel"/>
        </set>
    </class>
```


### Query 查询

```java
public List<GoodsTypeModel> getBySmUuidAndGmUuid(Long supplierUuid,
            Long[] goodsUuids) {
        String hql = "from SupplierModel sm join sm.gtms gtm join gtm.gms gm " +
                "where sm.uuid = :uuid and gm.uuid not in :uuids";
        Session s = this.getHibernateTemplate().getSessionFactory().getCurrentSession();
        Query q = s.createQuery(hql);
        q.setLong("uuid", supplierUuid);
        q.setParameterList("uuids", goodsUuids);
        return q.list();
    }
```

```java
public List<OrderModel> getByType(Set<Integer> orderStateSet) {
        String hql = "from OrderModel o where o.orderState in :ids";
        Session s = this.getHibernateTemplate().getSessionFactory().getCurrentSession();
        Query q = s.createQuery(hql);
        q.setParameterList("ids",orderStateSet);
        return q.list();
    }
```