[TOC]

MYSQL安装

安装位置有两个.

数据库服务位置: MySQL Server  
数据库数据文件位置: Server data files

UTF-8 在数据库中,叫utf8;


MYSQL中预设管理员名称为 root
---------------
SQL

    DDL  Data Definition Language  || create alter drop
    DML  Data Manipulation Language || insert update delete 
    DCL  Data Control Language || grant rollback commit.
    DQL  Data Query Language || select

连接数据库服务的命令

`mysql -u root -p`
根据提示输入密码 ,即可建立连接.

------------------------------
## DDL 数据库定义语言(库的操作)


1.创建一个库

```sql
create database day15 character set utf8 collate utf8_bin;  
```
------------------------------

2.显示mysql中都有哪些库了

```sql  
show databases;
```

3.删除一个数据库
```sql
drop database day15;
```
4.修改数据库码表和字符校对

```sql
alter database day15 character set utf8 collate utf8_bin;
```

5.当前要使用的库

```sql
use day15;
```

------------------------------
### 数据库中的数据类型

<span class='d_green'>ps</span>:了解,知道在什么情况下应该使用什么类型的数据

    数字型
        整型
            TINYINT    1字节    byte
            SMALLINT   2字节    short
            MEDIUMINT  3字节    
            **INT        4字节    int
            BIGINT     8字节  long
        浮点型
            FLOAT   单精度4字节  float
            DOUBLE  8字节      double
            DECIMAL 没有精度损失 根钱相关的数值.
    ------------------------------------------------        
    字符串类型  
        注意: 字符串类型要使用单引号包裹.
        短字符串类型
            CHAR/VARCHAR  (最大长度255字节)        String
    ------------------------------------------------        
    长字符串类型
        TEXT      保存文本(字符流) --> 当要保存的内容超过255字节时使用.  Writer
        BLOB      保存字节(字节流) --> 开发中用不到      Stream
        
    日期和时间类型
        date   只记录日期    1998-07-01
        time   只记录时间    14:30:00
        year   只记录年      2014
        datatime  又记录日期 又记录 时间    1998-07-01 14:30:00
        **timestamp 同上 1998-07-01 14:30:00  
        
<span class='d_red'>问题1</span>:以上两种 有什么区别?

    char(定长字符串),varchar(变长字符串).
    加入为char类型的指定数据长度为20. "abc" ==>  "abc                          "
    加入varchar类型指定长度为20, "abc" ==> "abc"

<span class='d_red'>问题2</span> datatime 和 timestamp 的区别?

    在保存上将,记录的内容是一样的.
    区别在于保存数据时, timestamp 可以不指定值, 那么系统会自动使用当前时间作为值.
    datatime 在保存时如果不指定 值, 那么将会插入一个null.
    所以开发中timestamp使用较多.

----------------------------------

### <span class='d_red'>与创建表相关的语句(DDL)</span> (记住,能够手写)
    CREATE TABLE table_name
    (
        field1  datatype  约束/主键约束 auto_increment,
        field2  datatype  约束,
        field3  datatype  约束
    )character set 字符集 collate 校对规则

1.创建表
```sql
    create table t_user(
        number int  primary key,
        name varchar(10) unique,
        password varchar(20) not null,
        sex  varchar(6) not null,
        email varchar(30) unique
    );
```

2.查看当前库中有哪些表
```sql    
    show tables;
```

3.查看表的结构
    
    desc 表名;

>+----------+-------------+------+-----+---------+-------+
>| Field    | Type        | Null | Key | Default | Extra |
>+----------+-------------+------+-----+---------+-------+
>| number   | int(11)     | YES  |     | NULL    |       |
>| name     | varchar(10) | YES  |     | NULL    |       |
>| password | varchar(20) | YES  |     | NULL    |       |
>| sex      | varchar(6)  | YES  |     | NULL    |       |
>| email    | varchar(30) | YES  |     | NULL    |       |
>+----------+-------------+------+-----+---------+-------+
        
+ 删除表

drop table 表名;


+ 添加一列
    
    alter table 表名 add 列名 类型;
    
    alter table t_user add salary double;
    
+ 修改列的类型
    
    alter table 表名 modify 列名 类型;
    
    alter table t_user modify sex char(6);
    
+ 修改列的名称
    
    alter table 表名 change  [column] old_col_name column_definition;
    
    alter table t_user change name firstname varchar(10);
    
+ 删除某列
    
    alter table 表名 drop 列名;
    
    alter table t_user drop email;
    
9.修改表的名称
    
    rename table 旧表名 to 新名;
    
    rename table t_user to t_student;
    
10(用的少)修改表的字符集. (如果创建表时不指定,默认使用数据库的字符集)
    
    alter table 表名 character set 字符集 collate 校对集;
    
    alter table t_student  character set utf8 collate utf8_bin;
    
            
//-------------------------------------------------------------------------------------------------         
列的约束 (掌握)   

    1.非空约束(not null)  指定非空约束的列, 在插入记录时 必须包含值.
    2.唯一约束(unique)  该列的内容在表中. 值是唯一的.
    3.主键约束(primary key)  当想要把某一列的值,作为该列的唯一标示符时,可以指定主键约束(包含 非空约束和唯一约束). 一个表中只能指定一个主键约束列.
                    主键约束 , 可以理解为 非空+唯一. 并且一张表中只能有一个主键约束.
            
    约束体现数据库的完整性.
        
    例如:创建带有约束的表
        create table t_student (
            number int(4) primary key,
            name varchar(10) unique,
            sex varchar(6) not null,
            age int(3) not null
        )
    
    
        
//----------------------------------------------------------------------------------------------------------------------------
主键自动增长 (掌握) (只在mysql 和 sql server中有该特性.)
    关键字: auto_increment
    首先主键自动增长只能用于整型数字的列.
    当使用该功能时, 在插入记录时, 无需对指定主键自增的列插入值. 数据库会默认为该列指定值, 指定的规则就是递增.
    1 , 2 ,3 ....
    注意: 该功能是mysql的功能,sql server 中也有. 其他数据库不具备该功能;
    
    例子: 创建 主键自增长列的表
    
    create table t_user(
        id int primary key auto_increment,
        name varchar(20) not null,
        email varchar(20) unique
    )
    
//--------------------------------创建修改表练习---------------------------------------------------

CREATE TABLE employee (
   id INT(10),
   NAME VARCHAR(10),
   gender VARCHAR(10),
   birthday DATETIME,
   entry_date TIMESTAMP,
   job VARCHAR(5),
   salary DOUBLE(5,3),
   RESUME TEXT
);


1在上面员工表的基础上增加一个image列。

    alter table employee add image blob;

2修改job列，使其长度为60。

    alter table employee modify job VARCHAR(60);

3删除gender列。

    alter table employee drop gender;

4表名改为user。

    rename table employee to user;

5修改表的字符集为utf8

    alter table user character set utf8 ;

6列名name修改为username

    alter table user change name username varchar(10);


//----------------------------------对表中数据的增删改-------------------------------------------------------------------
create table t_user(
        id int primary key auto_increment,
        name varchar(20) not null,
        email varchar(20) unique
    )
//-----------------------------------------

为表添加记录 (必须掌握)

insert into 表名[(列名1,列名2...)] values (值1,值2...);
        
1.插入一条数据

    1>指定要插入那些列
        insert into t_user(id,name,email) values(null,'tom','tom@itcast.cn'); 
        insert into t_user(id,email) values(null,'tom2@itcast.cn'); 实验非空约束
        insert into t_user(id,name,email) values(null,'jerry','tom@itcast.cn'); 实验唯一约束
    2>不指定插入哪些列, 需要指定每一列的值
        insert into t_user values(null,'jack','jack@itcast.cn'); 
        insert into t_user values(null,'肉丝','rose@itcast.cn'); 
        
 SHOW VARIABLES LIKE '%character%';    ==> 查看字符编码配置


| character_set_client     | gbk   客户端的编码
           |
| character_set_connection | utf8   客户端连接的编码
           |
| character_set_database   | utf8   数据库默认使用的编码
           |
| character_set_filesystem | BINARY  文件系统存放时使用的编码
           |
| character_set_results    | gbk    结果集的编码
           |
| character_set_server     | utf8    服务器编码
           |
| character_set_system     | utf8    内部系统编码
    
    
结论:     如果使用cmd 命令控制台操作 数据库, 
注意character_set_client 和  character_set_results  需要设置成GBK, 因为我们的命令控制航使用gbk码表显示中文.
使用如下命令设置:
方式1: 每次重新连接数据库都要重新设置.
set  character_set_client=gbk
set  character_set_results=gbk
方式2: 永久性修改 (不推荐)
修改my.ini
    C:\Program Files (x86)\MySQL\MySQL Server 5.5\my.ini
修改
        [client]

        port=3306

        [mysql]
        default-character-set=gbk
    但是使用方式2 修改, 会造成 整个数据库 在连接客户端时 使用的编码 都为gbk.(一劳永逸).影响范围比较大
    而相比方式1, 每次修改仅仅影响的是当前的连接. 只在当前连接内有效.
    
//--------------------------------------------------------------------------

修改一条记录 (必须掌握)

    update 表名 set 列名1 = 值 , 列名2 = 值 ....[where 条件1,条件2...]

1.修改表中id为3 的记录, 将name修改为rose;

    update t_user set name='rose' where id=3;
    

错误:  在修改的时候 不要忘记填写修改的条件, 不然将修改整张表的数据.
    
    update t_user set name='rose';

    
//----------------------------------------------------------------------------------------------------- 
    create table employee (
   id int,
   name varchar(20),
   gender varchar(20),
   birthday date,
   entry_date date,
   job varchar(30),
   salary double,
   resume longtext
);

insert into employee values(1,'zs','male','1980-12-12','2000-12-12','coder',4000,null);
insert into employee values(2,'ls','male','1983-10-01','2010-12-12','master',7000,null);
insert into employee values(3,'ww','female','1985-03-08','2008-08-08','teacher',2000,null);
insert into employee values(4,'wu','male','1986-05-13','2012-12-22','hr',3000,null);    
    

要求
将所有员工薪水修改为5000元。

    update employee set salary=5000 ;

将姓名为’zs’的员工薪水修改为3000元。

    update employee set salary=3000 where name='zs';


将姓名为’ls’的员工薪水修改为4000元,job改为ccc。

    update employee set salary=4000, job = 'ccc' where name='ls';

将wu的薪水在原有基础上增加1000元。

    update employee set salary = salary+1000 where name='wu';


//------------删除表记录相关-------------------------------------------------------------------------------------------------------------
删除记录语句 (必须掌握)

DELETE FROM   表名  WHERE 条件

删除表中名称为’zs’的记录。

    delete from employee where name = 'zs';


删除表中所有记录。

    delete from employee;

使用truncate删除表中记录。

    truncate table employee; 

DELETE 删除 和 TRUNCATE删除(了解) 两者有什么区别?

    
    1. 功能上来讲 truncate  和 delete from employee; 是一样的,都是将表中所有数据都删除.
    
    2.delete可以看做是将表中的数据逐行标记为删除. 
      truncate 直接将表内容表结构 一系列都物理删除. 然后重新创建表结构.
    3. delete 属于 DML语句.
       truncate 属于 DDL语句.
    4. delete删除数据可以恢复,但是删除不能释放硬盘空间.
        truncate 不能恢复.可以释放硬盘空间.
    

//-------------------------以上就是 增加 修改 删除 表记录 相关语句 ,(DML)-----------------------------------------------------------

//----------------------------------------------------------------------------------------------------------------------------------

DQL语句(DML) 查询语句.  (必须掌握)

语法：
  SELECT selection_list /*要查询的列名称*/
  FROM table_list /*要查询的表名称*/
  WHERE condition /*行条件*/
  GROUP BY grouping_columns /*对结果分组*/
  HAVING condition /*分组后的行条件*/
  ORDER BY sorting_columns /*对结果分组*/
  LIMIT offset_start, row_count /*结果限定*/

  
  
  
  CREATE TABLE stu (
    sid CHAR(6),
    sname       VARCHAR(50),
    age     INT,
    gender  VARCHAR(50)
);
INSERT INTO stu VALUES('S_1001', 'liuYi', 35, 'male');
INSERT INTO stu VALUES('S_1002', 'chenEr', 15, 'female');
INSERT INTO stu VALUES('S_1003', 'zhangSan', 95, 'male');
INSERT INTO stu VALUES('S_1004', 'liSi', 65, 'female');
INSERT INTO stu VALUES('S_1005', 'wangWu', 55, 'male');
INSERT INTO stu VALUES('S_1006', 'zhaoLiu', 75, 'female');
INSERT INTO stu VALUES('S_1007', 'sunQi', 25, 'male');
INSERT INTO stu VALUES('S_1008', 'zhouBa', 45, 'female');
INSERT INTO stu VALUES('S_1009', 'wuJiu', 85, 'male');
INSERT INTO stu VALUES('S_1010', 'zhengShi', 5, 'female');
INSERT INTO stu VALUES('S_1011', 'xxx', NULL, NULL);
//---------------------------------------------------------------

CREATE TABLE emp(
    empno       INT,
    ename       VARCHAR(50),
    job     VARCHAR(50),
    mgr     INT,
    hiredate    DATE,
    sal     DECIMAL(7,2),
    comm        DECIMAL(7,2),
    deptno      INT
);
INSERT INTO emp VALUES(7369,'SMITH','CLERK',7902,'1980-12-17',800,NULL,20);
INSERT INTO emp VALUES(7499,'ALLEN','SALESMAN',7698,'1981-02-20',1600,300,30);
INSERT INTO emp VALUES(7521,'WARD','SALESMAN',7698,'1981-02-22',1250,500,30);
INSERT INTO emp VALUES(7566,'JONES','MANAGER',7839,'1981-04-02',2975,NULL,20);
INSERT INTO emp VALUES(7654,'MARTIN','SALESMAN',7698,'1981-09-28',1250,1400,30);
INSERT INTO emp VALUES(7698,'BLAKE','MANAGER',7839,'1981-05-01',2850,NULL,30);
INSERT INTO emp VALUES(7782,'CLARK','MANAGER',7839,'1981-06-09',2450,NULL,10);
INSERT INTO emp VALUES(7788,'SCOTT','ANALYST',7566,'1987-04-19',3000,NULL,20);
INSERT INTO emp VALUES(7839,'KING','PRESIDENT',NULL,'1981-11-17',5000,NULL,10);
INSERT INTO emp VALUES(7844,'TURNER','SALESMAN',7698,'1981-09-08',1500,0,30);
INSERT INTO emp VALUES(7876,'ADAMS','CLERK',7788,'1987-05-23',1100,NULL,20);
INSERT INTO emp VALUES(7900,'JAMES','CLERK',7698,'1981-12-03',950,NULL,30);
INSERT INTO emp VALUES(7902,'FORD','ANALYST',7566,'1981-12-03',3000,NULL,20);
INSERT INTO emp VALUES(7934,'MILLER','CLERK',7782,'1982-01-23',1300,NULL,10);


CREATE TABLE dept(
    deptno      INT,
    dname       VARCHAR(14),
    loc     VARCHAR(13)
);

INSERT INTO dept VALUES(10, 'ACCOUNTING', 'NEW YORK');
INSERT INTO dept VALUES(20, 'RESEARCH', 'DALLAS');
INSERT INTO dept VALUES(30, 'SALES', 'CHICAGO');
INSERT INTO dept VALUES(40, 'OPERATIONS', 'BOSTON');


1.1　查询所有列 

select * from stu;  
select sid, sname, age, gender from stu;
使用* 相当于 查看所有列. 使用* 时,效率要比 输入指定列要低.  
    
1.2　查询指定列

select sid, sname, age, gender from stu;
    

2.1　条件查询介绍
条件查询就是在查询时给出WHERE子句，在WHERE子句中可以使用如下运算符及关键字：
    ?   =、!=、<>、<、<=、>、>=；
    ?   BETWEEN…AND；
    ?   IN(SET)；
    ?   IS NULL；
    ?   AND；
    ?   OR；  
    ?   NOT； 
2.2　查询性别为女，并且年龄小于50的记录

    select * from stu where gender='female' and age < 50;

2.3　查询学号为S_1001，或者姓名为liSi的记录

    SELECT * from stu where sid = 'S_1001' or sname='liSi';

    数据库中,sql语句不区分大小写 ,但是 数据区分大小写.

2.4 查询学号为S_1001，S_1002，S_1003的记录

    SELECT * from stu where sid = 'S_1001' or sid = 'S_1002' or sid = 'S_1003';
    
    select * from stu where sid in ('S_1001','S_1002','S_1003');
    
    以上两个语句一模一样,仅仅是写法的区别.使用运算符不同.
 
2.5　查询学号不是S_1001，S_1002，S_1003的记录
 
    select * from stu where sid not in ('S_1001','S_1002','S_1003');
    
    SELECT * from stu where not (sid = 'S_1001' or sid = 'S_1002' or sid = 'S_1003');
  
2.6　查询年龄为null的记录  
    
    SELECT * from stu where  age = null;
  
    注意: 数据库中, null不等于 null;
    
    
   SELECT * from stu where  age is null;
   
   使用"is" 判断即可.
   
2.7 查询年龄在20到40之间的学生记录

    SELECT * from stu where age between 20 and 40;
 
2.8查询性别非男的学生记录

    SELECT * from stu where not gender = 'male'; 
    SELECT * from stu where  gender != 'male'; 
    SELECT * from stu where  gender <> 'male'; 
    SELECT * from stu where  gender not in ('male'); 
    
2.9 查询姓名不为null的学生记录

    SELECT * from stu where sname is not null;
//--------------------------------------------------------------------------------------------------
where 字段 like '表达式'; 
% => 通配 通配任意个字符.
_ => 通配 通配单个字符.
说明: LIKE 条件后 根模糊查询表达式,  "_"==> 代表一个任意字符


3.1查询姓名由5个字母构成的学生记录

    select * from stu where sname like '_____';
    

3.2查询姓名由5个字母构成，并且第5个字母为“i”的学生记录

    select * from stu where sname like '____i';

3.3 查询姓名以“z”开头的学生记录
说明: "%"该通配符匹配任意长度的字符.
    
select * from stu where sname like 'z%';

3.4查询姓名中第2个字母为“i”的学生记录

select * from stu where sname like '_i%';

3.5　查询姓名中包含“a”字母的学生记录

select * from stu where sname like '%a%';
//-----------------------------------------------------------------
4.1　去除重复记录
    关键词: distinct => 去除重复记录.
    
    select distinct gender from stu;
        

4.2查看雇员的月薪与佣金之和

    select ename,sal+comm from emp;
    
    注意: 任何值与null 计算结果都是null; 使用如下写法:
    使用 IFNULL(参数1,参数2) 函数.  => 判断参数1的值是否是null, 如果是null ,返回参数2.
    select ename,sal+ifnull(comm,0) from emp;

4.3　给列名添加别名

    select ename,sal+ifnull(comm,0) from emp;
    //下面是加别名的写法
    select ename,sal+ifnull(comm,0) as '实际收入' from emp;
    select ename '姓名',sal+ifnull(comm,0) '实际收入' from emp;
//------------------------------------------------------------------------------------------------------------------------------


5.1　查询所有学生记录，按年龄升序排序
    asc: 升序
    desc:降序

    select * from stu order by age ;  ==> 不写升序 降序, 默认是升序
    select * from stu order by age asc;
如果不指定排序规则,则使用升序排列

5.2 查询所有学生记录，按年龄降序排序

    select * from stu order by age desc;
    
    null: 是最小的.

5.3 查询所有雇员，按月薪降序排序，如果月薪相同时，按编号升序排序

    select * from emp order by sal desc,empno asc;


聚合函数
聚合函数是用来做纵向运算的函数：
?   COUNT()：统计指定列不为NULL的记录行数；
?   MAX()：计算指定列的最大值，如果指定列是字符串类型，那么使用字符串排序运算；
?   MIN()：计算指定列的最小值，如果指定列是字符串类型，那么使用字符串排序运算；
?   SUM()：计算指定列的数值和，如果指定列类型不是数值类型，那么计算结果为0；
?   AVG()：计算指定列的平均值，如果指定列类型不是数值类型，那么计算结果为0；

6.1　COUNT
当需要纵向统计时可以使用COUNT()。
?   1>查询emp表中记录数：

        select count(*) from emp;
    
    2>查询emp表中有佣金的人数：
        
        select count(comm) from emp where comm != 0;
        
        注意: 在计数时 null 不计入.
    
    3>查询emp表中月薪大于2500的人数：
    
        select count(*) from emp where sal > 2500;
    
    4>统计月薪与佣金之和大于2500元的人数：
        
        select count(*) from emp where (sal+ifnull(comm,0)) > 2500;
        
    5>查询有佣金的人数，以及有领导的人数：

        select count(*) from emp where comm is not null and comm>0 and mgr is not null;

6.2　SUM(计算总和)和AVG(计算平均值)
    当需要纵向求和时使用sum()函数。
?   1>查询所有雇员月薪和：
    
        select sum(sal) from emp;
    
    2>查询所有雇员月薪和，以及所有雇员佣金和：

        
        select sum(sal),sum(comm) from emp;
    
    
    3>查询所有雇员月薪+佣金和：
    
        select sum(sal+ifnull(comm,0)) from emp;
    
    4>统计所有员工平均工资：
    
        select avg(sal) from emp;
    
6.3　MAX和MIN
?   查询最高工资和最低工资：

    select max(sal),min(sal) from emp;
    
//---------------------------------------------------------------------------------------------------------------------------------------

    
分组查询

当需要分组查询时需要使用GROUP BY子句，例如查询每个部门的工资和，这说明要使用部分来分组。



?   1>查询每个部门的部门编号和每个部门的工资和：

            
            select  deptno , sum(sal)  from emp group by  deptno;
        

    2>查询每个部门的部门编号以及每个部门的人数：
    
            select deptno , count(empno) from emp group by deptno;


    3>查询每个部门的部门编号以及每个部门工资大于1500的人数：
    
            select deptno , count(empno) from emp where sal > 1500 group by deptno;

HAVING子句
?   查询工资总和大于9000的部门编号以及工资和：
    
    select deptno,sum(sal) from emp  group by deptno having sum(sal) > 9000;
    
    
 HAVING过滤 和 where过滤区别?
        HAVING ==> 在分组后 进行过滤筛选.
        where ==> 在分组之前 进行过滤筛选.
    
    注意: 如果 使用 having 或 where 都可以完成需求, 那么一定要优先使用 where.
            因为 分组 本身资源消耗极大.

//-----------------------------------------------以上是查询---------------------------------------------------------------------------------


//----------------------------------------------一下是分页相关知识---------------------------------------------------------------------------------------
LIMIT（MySQL方言） (必须掌握)
LIMIT用来限定查询结果的起始行，以及总行数。

1>查询5行记录，起始行从0开始

select * from emp order by sal desc limit 0,5;

2>　查询10行记录，起始行从3开始

select * from emp limit 3,10;

3>如果一页记录为5条，希望查看第3页记录应该怎么查呢？
?   第一页记录起始行为0，一共查询5行；
    select * from emp order by sal desc limit 0,5;
?   第二页记录起始行为5，一共查询5行；
    select * from emp order by sal desc limit 5,5;
?   第三页记录起始行为10，一共查询5行；
    select * from emp order by sal desc limit 10,5;