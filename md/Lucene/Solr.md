[TOC]

## Solr

### solr 简介
    Solr也是Apache旗下的顶级项目。基于lucene开发的全文检索服务器。
    Solr和Lucene的版本同步更新。
    solr可以独立运行，需要运行在web容器中。例如：Tomcat、WebLogic、jetty、Jboss
    solr服务对外提供服务，可以使用服务的接口和solr服务进行交换。
    使用get和post方法基于http的请求，和客户端进行数据交互。
    传递的数据可以是xml也可以是json。

![](../pic/solr_01.png)

-----------------------------------

##### 下载地址
[solr 下载地址](http://lucene.apache.org/solr/)
>__要求:__
`Jdk`：1.7以上
`Tomcat` 7.0以上


-----------------------------------
### Solr的部署步骤：

1. 安装Tomcat
2. 把`solr-4.10.3\dist\solr-4.10.3.war`放到tomcat的webapp下，改名为solr.war
3. 解压war包。
4. 复制<span class='d_red'>`\solr-4.10.3\example\lib\ext`</span>目录下的jar复制到solr工程的lib下。主要是日志相关的jar包
5. 在solr项目的WEB-INF下创建一个classes目录，把<span class='d_red'>`\solr-4.103\example\resources\log4j.properties`</span>复制过去。
6. 需要配置__SolrHome__及__SolrCore__
    * __solrHome__：就是solr服务器配置文件的目录。`\solr-4.10.3\example\solr`文件夹就是一个标准的solrhome。
    * __SolrCore__：一个SolrCore就是一个完整功能的索引库。可以类比mysql数据库的一个库。索引库之间数据是隔离的。
    * 把<span class='d_red'>`\solr-4.10.3\example\solr`</span>文件夹复制到指定位置（任意位置都可以）。本例中`D:\1125\solr`目录下，改名为solrhome（改名不是必须，便于理解）
7. 配置`solrconfig.xml`("solrhome"`\collection1\conf`)（此步骤也不是必须的）
    + `<lib>`：solrcore需要引用的扩展包。默认的路径是__SolrCore__的目录也就是__collection1__目录下的lib文件夹。（此步骤也不是必须的）
    + `<dataDir>`：索引库的存放位置。默认路径是__SolrCore__的目录也就是__collection1__目录下的data文件夹。如果想使用默认的路径不需要修改配置。data文件夹会自动创建。
    + `<requestHandler>`:配置solr服务的接口。其中索引的维护使用`/update`   查询：`/query` 、`/select`
8. 在solr工程的web.xml中配置solrhome的位置。目的是让solr服务器知道solrhome的位置。
```xml
<!--配置jndi告诉solr工程我们的solrhome的位置-->
    <env-entry>
        <env-entry-name>solr/home</env-entry-name>
        <env-entry-value>D:/1125/solr</env-entry-value>
        <env-entry-type>java.lang.String</env-entry-type>
    </env-entry>
```
9. 启动tomcat。

##### 查看solr服务
访问：`http://localhost:8080/solr`

-----------------------------------

### schema
* 索引的维护包含：增删改操作。

- 创建索引：

    <d_c class='d_red'>id</d_c>域是必须的，是一个唯一的<d_c class='d_red'>key</d_c>，相当于关系型数据库表的主键。
    域的名字必须是先定义后使用，不能随意起名。

+ 域的名字在哪儿定义？

    `solrcore\conf\scheam.xml`中定义

##### schema.xml文件

<d_c class='d_red'>field</d_c>：在标签中定义了

     name：域的名字
     index：是否索引
     stored：是否存储
     multiValued：是否多值
     type：域的数据类型，是否分词取决于数据类型

<d_c class='d_red'>dynamicField</d_c>：

    动态域，只要域名和动态域的表达式相匹配，此域名就可以使用。

<d_c class='d_red'>uniqueKey</d_c>： 

    `<uniqueKey>id</uniqueKey>`定义了主键域。

<d_c class='d_red'>copyField</d_c>：

    当复制域的source指定域存储时，自动复制到目标域dest指定的域中。

<d_c class='d_red'>fieldType</d_c>：域的类型

    name：类型名
    class：对应类型的一个类。是否分词取决于此类。
在域的类型中配分析器。

-----------------------------------

## <d_c class='d_red'>配置中文分析器</d_c>
1. 把`IKAnalyzer2012FF_u1.jar`复制到<d_c class='d_green'>solr工程的lib</d_c>下。
1. 把配置文件及扩展词典和停用词词典复制到<d_c class='d_green'>solr工程的classes目录下</d_c>。一定要注解扩展词典及停用词词典的编码格式要为utf-8.
1. 修改`schema.xml`
1）增加一个`FieldType`
```xml
    <!-- IKAnalyzer-->
    <fieldType name="text_ik" class="solr.TextField">
      <analyzer class="org.wltea.analyzer.lucene.IKAnalyzer"/>
    </fieldType>
```
2）添加域`field`，指定field的type属性为<d_c class='d_red'>text_ik</d_c>
```xml
<!--IKAnalyzer Field-->
   <field name="title_ik" type="text_ik" indexed="true" stored="true" />
   <field name="content_ik" type="text_ik" indexed="true" stored="true" multiValued="true"/>
```
1. 重启tomcat

-----------------------------------

## <d_c class='d_red'>业务字段配置</d_c>
```xml
<!--product-->
//是和数据库中表的字段对应
   <field name="product_name" type="text_ik" indexed="true" stored="true"/>
   <field name="product_price"  type="float" indexed="true" stored="true"/>
   <field name="product_description" type="text_ik" indexed="true" stored="false" />
   <field name="product_picture" type="string" indexed="false" stored="true" />
   <field name="product_catalog_name" type="string" indexed="true" stored="true" />
 

//创建一个默认搜索域
   <field name="product_keywords" type="text_ik" indexed="true" stored="false" multiValued="true"/>


//把商品名称和商品描述的信息，复制到默认搜索域中。   
    <copyField source="product_name" dest="product_keywords"/>
    <copyField source="product_description" dest="product_keywords"/>
```

-----------------------------------

### <span class='d_red'>配置dataimport插件</span>
1. dataimport插件相关的jar复制到`solrcore\lib`下
1. 把mysql的jar包一并复制到`solrcore\lib`下
![](../pic/dataimport.png)
1. 修改`solrconfig.xml`文件("solrhome"`\collection1\conf`)
* 增加一个`requesthandler`
```xml
  <requestHandler name="/dataimport" class="org.apache.solr.handler.dataimport.DataImportHandler">
    <lst name="defaults">
      <str name="config">data-config.xml</str>
    </lst>
  </requestHandler>
```
* 在`solrcore\conf`(对应`\collection1\conf`)目录下创建一个data-config.xml文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>  
<dataConfig>   
<dataSource type="JdbcDataSource"   
          driver="com.mysql.jdbc.Driver"   
          url="jdbc:mysql://localhost:3306/solr"   
          user="root"   
          password="123"/>   
<document>   
    <entity name="product" query="SELECT pid,name,catalog_name,price,description,picture FROM products ">
         <field column="pid" name="id"/> 
         <field column="name" name="product_name"/> 
         <field column="catalog_name" name="product_catalog_name"/> 
         <field column="price" name="product_price"/> 
         <field column="description" name="product_description"/> 
         <field column="picture" name="product_picture"/> 
    </entity>   
</document>   

</dataConfig>
```
1. 重启tomcat

-----------------------------------

### 索引库的修改

    如果想修改id为1的文档，直接向索引库中添加一个id为1 的文档，新的文档会把原理的文档替换掉。
    原理就是先删除id为1的文档，然后又新追加了一个文档。

#### 索引库的删除

1. 直接指定id来删除
先选择Document Type为xml，然后在Document(s)区域中写入：
```xml
<delete>
<id>2</id>
</deleter>
<commit/> <!--提交-->
```
就可以删除指定id的文档。

1. 可以指定查询条件来删除：
在solr中对lucene的查询语法完全支持。
```xml
<delete>
<query>*:*</query><!--指定查询条件，删除所有文档-->
</delete>
<commit/> 
```

-----------------------------------

## Request-Handler (qt)：
<span class='d_green'>/select</span> 在`solrconfig.xml`中定义的。

* <span class='d_red'>q:查询条件</span>

>查询的语法：
>>      域名：查询条件
>>      例如：product_name:花儿朵朵、*:*

>范围查询语法：

>>       field:[* TO 100]    小于等于100
>>       field:[100 TO *]  大于等于100
>>       field:[100 TO 200] 大于等于100小于等于200

组合条件查询：

    product_price:[0 TO 1] OR product_catalog_name:时尚卫浴


* <span class='d_red'>fq</span>
    
        过滤条件，过滤条件和查询条件的区别是，过滤条件是在查询条件
        查询出来的结果基础上进行过滤。

>ps: 过滤条件可以是多个。

* <span class='d_red'>sort</span>

        排序条件，多个排序条件使用“,”分隔
        域名 asc|desc
        例如：product_price desc,product_name asc

* <span class='d_red'>start, rows</span>
分页信息
>start：
    
        起始的记录

* <span class='d_red'>rows</span>
    
        每页显示的总条数


* <span class='d_red'>fl</span>
    
        返回结果中域的列表，如果不指定默认返回所有域。多个域使用“,”分隔。

* <span class='d_red'>df</span>
    
        默认搜索域

* <span class='d_red'>wt</span>
    
        返回结果的格式xml、json

* <span class='d_red'>hl</span>

-----------------------------------

## Solr的客户端：SolrJ

__如何把solrJ集成到项目中。也就是如何使用SolrJ调用solr服务。__

<span class='d_green'>实现步骤</span>

    1. 创建一个java工程
    2. 导入solrJ的jar包。
    \solr-4.10.3\dist\solr-solrj-4.10.3.jar
    \solr-4.10.3\dist\solrj-lib下所有的jar包。
    还需要\solr-4.10.3\example\lib\ext下所有的jar包。
    三部分组成。
    3. SolrServer对象来和服务端建立连接。HttpSolrServer创建SolrServer对象。
    4. 使用solrJ对solr服务端进行增删改查操作。

#### 添加文档：
1. 创建一个SolrInputDocument对象
1. 向document对象中添加域
1. 向服务端添加文档
1. commit。

```java
    //向solr索引库中添加文档
    @Test
    public void addDocument() throws Exception {
        
        //创建连接
        SolrServer server = new HttpSolrServer("http://localhost:8080/solr");
        //创建一个文档
        SolrInputDocument document = new SolrInputDocument();
        //向文档中添加域
        document.addField("id", "a001");
        document.addField("title_ik", "使用SolrJ新添加的文档");
        //向索引库中添加文档
        server.add(document);
        //提交
        server.commit();
        
    }
```

-----------------------------------

#### 删除文档：
deleteById
deleteByQuery
更新文档：操作同添加文档。

```java
    public void deleteDocument() throws SolrServerException, IOException {
        //创建连接
        SolrServer server = new HttpSolrServer("http://localhost:8080/solr");
//      server.deleteById("a001");
//      server.deleteByQuery("id:3");
        server.deleteByQuery("*:*");
        server.commit();
    }
```

-----------------------------------

#### <span class='d_red'>查询</span>

```java
public void queryIndex() throws Exception {
    //创建连接
    SolrServer server = new HttpSolrServer("http://localhost:8080/solr");
    //创建一个查询对象
    SolrQuery query = new SolrQuery();
    //设置查询条件
    query.setQuery("麻布");
    //设置过滤条件
    query.addFilterQuery("product_catalog_name:幽默杂货");
    //设置排序条件
    query.setSort("product_price", ORDER.asc);
    //分页信息
    query.setStart(0);
    query.setRows(20);
    //设置返回结果中包含的域,不设置默认返回所有域
    query.setFields("id", "product_name", "product_price", "product_catalog_name");
    //设置默认搜索域
    query.set("df", "product_keywords");
    //高亮显示
    query.setHighlight(true);
    //高亮的域
    query.addHighlightField("product_name");
    //高亮前缀
    query.setHighlightSimplePre("<em>");
    //后缀
    query.setHighlightSimplePost("</em>");
    //执行查询
    QueryResponse result = server.query(query);
    //返回结果的列表
    SolrDocumentList solrDocumentList = result.getResults();
    System.out.println("共查找到文档：" + solrDocumentList.getNumFound());
    //遍历结果
    for (SolrDocument solrDocument:solrDocumentList) {
        //取高亮显示
        Map<String, Map<String, List<String>>> highlighting = result.getHighlighting();
        
        String productName = "";
        //如果高亮结果中没有高亮信息
        if (highlighting.get(solrDocument.get("id")) == null) {
            productName = (String) solrDocument.get("product_name");
        } else {
            //有高亮信息
            productName= highlighting.get(solrDocument.get("id")).get("product_name").get(0);
        }
        
        System.out.println(solrDocument.get("id") + "\t" 
                        + productName + "\t" 
                        + solrDocument.get("product_price") + "\t" 
                        + solrDocument.get("product_catalog_name"));
        
        
    }
}
```

-----------------------------------



-----------------------------------

-----------------------------------



-----------------------------------












