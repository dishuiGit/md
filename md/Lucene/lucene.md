
[TOC]

### 全文检索的应用场景
- 搜索引擎
    + asdfasdf
- 站内搜索
- 电商网站搜索商品
- eclipse的帮助文档

----------------

### 案例：从文件中查找，找出文件内容包含spring字样的文件。
方案一：目测
方案二：顺序扫描，效率很低。
方案三：先把文章读取到程序里，根据空格对文章的内容进行分割（分词）形成一个词语列表。在查找内容时，直接去列表查找。找到列表中的词后，根据这个词就能找到相应的文章。
这个过程就叫做全文检索的过程。词语列表就是索引。类似于查字典的过程。

----------------------

**PS:**
>   Lucene和搜索引擎的区别：
>   Lucene是实现全文检索的API
>   搜索引擎：一套独立运行的搜索系统。


### Lucene实现全文检索的流程
[![](../pic/lucene流程图.png)](pic/lucene流程图.png "Lucene检索流程图")

#### 创建索引：

**第一步：**获得原始文档，原始文档就是文件夹中的一堆文件。可以使用文件流来读取文件的内容。

>   + 知识扩展：
>       * 搜索引擎的原始文档：互联网上所有的网页。
>       * 获得原始文档需要使用网络爬虫。
>       * Nutch：apache的开源爬虫项目。
>       * Heritrx：开源的爬虫项目。
>       * soup：解析html的工具包。
>
>电商网站的原始文档：数据的记录。
>solr工具有一个插件可以直接将数据库中记录导入到索引库中。dataimport插件。


**第二步：**创建文档对象
>* 在Lucene里有一个类Document。就可以创建一个文档对象。一个文件就对应一个文档对象。
>* 为了对文件进行清晰的描述，需要有多个域（Field）来存储不同的内容例如：文件名、文件内容、文件的大小、文件的路径。
>* 一个document对象可以理解为关系数据库中的一条记录，域（field）可以理解为一条记录的多个字段。



第三步：文档的分词：
分词时需要指定分词的域。也就是指定哪个域的内容需要进行分词。本例来说就可以根据空格来分词可以得到一个词汇列表。同时记录词语的出处。同一个词要合并。
0、先对文档指定分词的域进行分词。
1、转换大小写，统一转换成小写
2、去除停用词，去除掉没有意义的词
3、去除标点符号。


第四步：创建索引

存储的内容分两部分：
1、根据分析出来的关键词创建的索引
2、Document对象。需要存储的域的是需要展示给用户的内容。如果不需要展示就可以不存储到索引库。

注意：文件的内容进行分词后创建索引，文件的内容可以不存储到document中。



:tm::tired_face:

## 3 Lucene 入门程序

#### 需求
    实现一个资源管理器的搜索功能，通过关键字搜索，凡是文件名或文件内容包括关键字的文件都要找出来。
    *注意*：该入门程序只对文本文件(.txt)搜索。

#### 开发环境

**Jdk**：`1.7.0_72`
**开发工具**：`eclipse indigo`

**Lucene包**：
>   `lucene-core-4.10.3.jar`
>   `lucene-analyzers-common-4.10.3.jar`
>   `lucene-queryparser-4.10.3.jar`
>   `commons-io-2.4.jar`
>   `junit-4.9.jar`

#### 创建数据源目录
>创建数据源目录F:\develop\lucene\searchsource ，该目录存放要搜索的原始文件。

#### 创建索引目录
>创建索引目录 F:\develop\lucene\indexdata，该目录存放创建的索引文件。

### Document和Field

    索引的目的是为了搜索，索引前要对搜索的内容创建成Document和Field。Document文档是Lucene搜索的单位，最终要将文档中的数据展示给用户，文档中包括多个Field，在设计文档对象时，可以将一个Document对应关系数据库的一条记录，字段对应Field，但是要注意：Document结构属于NOSql（非关系），不同的Document包括不同的Field，建议将相同类型数据的Document保持Field一致，方便管理维护，避免将不同类型Field数据融合在一个Document中。
    比如：
    文件信息Document，Field包括：文件名称、文件类型、文件大小、文件路径、文件内容等，避免加入非文件信息Field。

#### Field属性

Field是文档中的域，包括Field名和Field值两部分，一个文档可以包括多个Field，Document只是Field的一个承载体，Field值即为要索引的内容，也是要搜索的内容。

1.是否分析(tokenized)
是：将Field值分析出语汇单元即作分词处理，将词进行索引
比如：商品名称、商品简介等，这些内容用户要输入关键字搜索，由于搜索的内容格式大、内容多需要分析后将语汇单元索引。

否：不作分词处理
比如：商品id、订单号、身份证号等 

2.是否索引(indexed)
是：将Field分析后的词或整个Field值进行索引，只有索引方可搜索到。
比如：商品名称、商品简介分析后进行索引，订单号、身份证号不用分析但也要索引，这些将来都要作为查询条件。

否：不索引无法搜索到
比如：商品id、文件路径、图片路径等，不用作为查询条件的不用索引。

3.是否存储(stored)
是：将Field值存储在文档中，存储在文档中的Field才可以从Document中获取
比如：商品名称、订单号，凡是将来要从Document中获取的Field都要存储。

否：不存储Field值，不存储的Field无法通过Document获取
比如：商品简介，内容较大不用存储。如果要向用户展示商品简介可以从系统的关系数据库中获取商品简介。

| Field类  | type |Analyzed | Indexed | Stored | 说明 |
| ------------- | ------------- | ------------- | ------------- | ------------- |
| StringField(FieldName, FieldValue,Store.YES)) | 字符串  | N  | Y  | Y或N  | 这个Field用来构建一个字符串Field，但是不会进行分析，会将整个串存储在索引中，比如(订单号,姓名等) 是否存储在文档中用Store.YES或Store.NO决定 |
| LongField(FieldName, FieldValue,Store.YES)  | Long型  | Y  | N  | Y或N | 这个Field用来构建一个Long数字型Field，进行分析和索引，比如(价格)是否存储在文档中用Store.YES或Store.NO决定 |
| StoredField(FieldName, FieldValue)  | 重载方法，支持多种类型  | N  | N  | Y  | 这个Field用来构建不同类型Field不分析，不索引，但要Field存储在文档中 |
| TextField(FieldName, FieldValue, Store.NO)或TextField(FieldName, reader)  | 字符串或流  | Y  | Y  | Y或N  | 如果是一个Reader, lucene猜测内容比较多,会采用Unstored的策略. |

### IndexWriter和Directory

IndexWriter是索引过程的核心组件，通过IndexWriter可以创建新索引、更新索引、删除索引操作。IndexWriter需要通过Directory对索引进行存储操作。

Directory描述了索引的存储位置，底层封装了I/O操作，负责对索引进行存储。它的子类常用的包括FSDirectory（在文件系统存储索引）、RAMDirectory（在内存存储索引）。


### 索引程序代码

>   <font color="red">分析：</font>
>   **确定文档的各各域的属性和类型：**
>   
>       * 文件名称：不分析、索引、存储，使用StringField方法
>       * 文件路径：不分析、不索引、存储，使用StoredField方法
>       * 文件大小：分析、索引、存储，使用LongField方法
>       * 文件内容：分析，索引，不存储，使用TextField方法

##### 创建索引
```java
public void createIndex() throws IOException{
        //指定索引库存放的路径,存放到磁盘
        Directory directory = FSDirectory.open(new File("d:/temp/1125"));
        //分析器
        Analyzer analyzer = new StandardAnalyzer();
        //配置indexWriter
        IndexWriterConfig config = new IndexWriterConfig(Version.LATEST, analyzer);
        IndexWriter indexWriter = new IndexWriter(directory, config);
        String serachPath = "D:/1125/JavaEE就业视频/Lucene&Solr/lucene/00.参考资料/searchsource";
        List<Document> docs = file2Document(serachPath);
        //写入索引库
        indexWriter.addDocuments(docs);
        //关闭indexWriter
        indexWriter.close();
    }
    //目录下所有file转 Document
    private List<Document> file2Document(String serachPath) throws IOException {
        List<Document> listDoc = new ArrayList<Document>();
        //原始文档路径
        File path = new File(serachPath);
        for(File file:path.listFiles()){
            //创建Document
            Document document = new Document();
            //创建域
            //文件名
            Field fieldTitle = new TextField("title", file.getName(), Store.YES);
            //文件内容
            String content = FileUtils.readFileToString(file);
            Field fieldContent = new TextField("content", content, Store.YES);
            //文件路径
            Field fieldPath = new StoredField("path", file.getPath());
            //文件大小
            Field fieldSize = new LongField("size", FileUtils.sizeOf(file), Store.YES);
            
            document.add(fieldTitle);
            document.add(fieldContent);
            document.add(fieldPath);
            document.add(fieldSize);
            
            listDoc.add(document);
        }
        return listDoc;
    }
```

##### 查询索引库

```java
    public void queryIndex() throws Exception {
        //创建一个查询
        //Term的构造方法的参数
        //第一个参数：是要查询的域
        //第二个参数：是要查询的关键字
//      Query query = new TermQuery(new Term("title", "java.txt"));
        Query query = new TermQuery(new Term("content", "数据库"));
        //执行查询  
        //索引库的路径
        Directory directory = FSDirectory.open(new File("d:/temp/1125"));
        //先创建一个Indexreader对象,需要知道索引库的路径
        IndexReader indexReader = DirectoryReader.open(directory);
        //再创建一个IndexSearcher
        IndexSearcher searcher = new IndexSearcher(indexReader);
        //执行查询
        //第一个参数：一个Query对象
        //第二个参数：查询结果返回条数的最大值。
        TopDocs topDocs = searcher.search(query, 2);
        //查看总共匹配的document的记录数
        System.out.println(topDocs.totalHits);
        //查看结果集
        for (ScoreDoc scoreDoc:topDocs.scoreDocs) {
            System.out.println("文档的编号：" + scoreDoc.doc);
            Document document = searcher.doc(scoreDoc.doc);
            System.out.println(document.get("title"));
            System.out.println(document.get("content"));
            System.out.println(document.get("size"));
            System.out.println(document.get("path"));
            System.out.println("==============郁闷的分割线==============");
            
        }
        
        indexReader.close();
    }
```

### 索引结构
:
    
Lucene的索引结构为倒排索引，倒排文件或倒排索引是指索引对象是文档或者文档集合中的单词等，
>用来存储这些单词在一个文档或者一组文档中的存储位置，是对文档或者文档集合的一种最常用的索引机制。

[![](../pic/lucene索引.png)](pic/lucene索引.png "Lucene索引流程图")

<font color='red'>__Lucene__</font>索引`index`由若干段(`segment`)组成，每一段由若干的文档（`document`）组成，
每一个文档由若干的域（`field`）组成，每一个域由若干的项（`term`）组成。
项是最小的索引单位，如果`Field`进行分析，可能会分析出多个语汇单元（词），
每个__词__就是一个<font color='blue'>__Term项__</font>，如果`Field`不进行分析，整个`Field`就是一个Term项。
__项__直接代表了一个字符串以及其在文件中的位置、出现次数等信息。<font color='blue'>__域__</font>将<font color='blue'>__项__</font>和<font color='blue'>__文档__</font>进行关联。

-----------------------------------

### 确定索引目录

>搜索是要从索引中查找，确定索引目录即上边创建的索引目录`F:\develop\lucene\indexdata`。

##### IndexSearcher和IndexReader 

>通过IndexSearcher执行搜索，构建IndexSearcher需要IndexReader读取索引目录，如下图：

![](../pic/lucene_indexSeracher.png)


<font color='red'>*需要注意：*</font>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;__`Indexreader`__打开需要耗费很大的系统资源，建议使用一个__`IndexReader`__，
如果索引进行添加、修改或删除需要打开新的`Reader`才可以搜索到。

###### IndexSearcher搜索方法如下：
>| __方法__ | __说明__  |
| :------------ |:---------------:|
| indexSearcher.search(query, n)      | 根据Query搜索，返回评分最高的n条记录 |
| indexSearcher.search(query, filter, n)      | 根据Query搜索，添加过滤策略，返回评分最高的n条记录        |
| indexSearcher.search(query, n, sort) | 根据Query搜索，添加排序策略，返回评分最高的n条记录        |
| indexSearcher.search(booleanQuery, filter, n, sort) | 根据Query搜索，添加过滤策略，添加排序策略，返回评分最高的n条记录 |

### TopDocs

__`Lucene`搜索结果可通过`TopDocs`遍历，`TopDocs`类提供了少量的属性，如下：__

>| 方法或属性  | 说明  |
| :------------ |:---------------:|
| <font color='green'>totalHits</font> | 匹配搜索条件的总记录数 |
| <font color='green'>scoreDocs</font> | 顶部匹配记录        |

*注意：*
>Search方法需要指定匹配记录数量n：`indexSearcher.search(query, n)`
+ `TopDocs.totalHits： `是匹配索引库中所有记录的数量
+ `TopDocs.scoreDocs：`匹配相关度高的前边记录数组，
+ `scoreDocs`的长度小于等于search方法指定的参数n

### Analyzer分析器

#### Analyzer使用时机
##### 索引时使用Analyzer
---------------------
>* 输入关键字进行搜索，当需要让该关键字与文档域内容所包含的词进行匹配时需要对文档
* 域内容进行分析，需要经过Analyzer分析器处理生成语汇单元（Token）。分析器分析的对
* 象是文档中的Field域。当Field的属性tokenized（是否分词）为true时会对Field值进行
分析，如下图：

>![](../pic/lucene_analyzer.png)

>__对于一些Field可以不用分析:__
1. 不作为查询条件的内容，比如__文件路径__
2. 不是匹配内容中的词而匹配Field的整体内容，比如__订单号__、__身份证号__等。

##### 搜索时使用Analyzer
---------------------
>- 对搜索关键字进行分析和索引分析一样，使用Analyzer对搜索关键字进行分析、
- 分词处理，使用分析后每个词语进行搜索。比如：搜索关键字：spring web ，
- 经过分析器进行分词，得出：spring  web拿词去索引词典表查找 ，
- 找到索引链接到`Document`，解析Document内容。
- 对于匹配整体Field域的查询可以在搜索时不分析，比如根据订单号、身份证号查询等。
- *<font color='red'>注意：</font>* 
    + **<font color='red'>搜索使用的分析器要和索引使用的分析器一致。</font>**

### Analyzer执行过程
##### 代码分析

    Analyzer是一个抽象类，在Lucene的lucene-analyzers-common包中提供了很多分析器，
    比如：org.apache.lucene.analysis.standard.standardAnalyzer标准分词器，
    它是Lucene的核心分词器，它对分析文本进行分词、大写转成小写、去除停用词、去除标点符号等操作过程。







:exclamation:
:green_heart:

| Left-Aligned  | Center Aligned  | Right Aligned |
| :------------ |:---------------:| -----:|
| col 3 is      | some wordy text | $1600 |
| col 2 is      | centered        |   $12 |
| zebra stripes | are neat        |    $1 |
































































| Year | Temperature (low) | Temperature (high) |  
| ---- | ----------------- | -------------------|  
| 1900 |               -10 |                 25 |  
| 1910 |               -15 |                 30 |  
| 1920 |               -10 |                 32 | 