[TOC]


#### 字符函数

```sql
	select lower('Hello World') 转小写,upper('Hello World') 转大写,
	       initcap('hello world') 首字母大写
	from dual;
```

	转小写      转大写      首字母大写   
	----------- ----------- -----------  
	hello world HELLO WORLD Hello World
-----------------------------------------------
#### substr(a,b) 从a中，第b位开始取，取右边所有的字符

```sql
	select substr('Hello World',3) from dual;
```

	SUBSTR('H 
	--------- 
	llo World 
-----------------------------------------------
#### substr(a,b,c) 从a中，第b位开始取，取c位

```sql
	select substr('Hello World',3,4) from dual;
```

	SUBS 
	---- 
	llo  

-----------------------------------------------
#### length 字符数  lengthb字节数

```sql
	select length('Hello World') 字符,lengthb('Hello World') 字节
	from dual;
```

	      字符       字节  
	---------- ----------  
	        11         11   

```sql
	select length('中国') 字符,lengthb('中国') 字节
	from dual
```

	      字符       字节   
	---------- ----------  
	         2          4 
-----------------------------------------------
#### instr(a,b): 在a中，查找b

```sql
	select instr('Hello World','ll')  位置  from dual;
```

	      位置    
	----------    
	         3 
-----------------------------------------------
#### lpad 左填充 rpad右填充
-- abcd  ---> 10位

```sql
	select lpad('abcd',10,'*') 左,rpad('abcd',10,'*') 右
	from dual;
```

	左         右           
	---------- ----------   
	******abcd abcd****** 
-----------------------------------------------
#### trim

```sql
	select trim('H' from 'Hello WorldH') from dual;
```

	TRIM('H'FR  
	----------  
	ello World 
-----------------------------------------------
#### replace

```sql
	select replace('Hello World','l','*') from dual;
```

	REPLACE('HE 
	----------- 
	He**o Wor*d
-----------------------------------------------
#### 四舍五入

```sql
	select round(45.926,2) 一,round(45.926,1) 二,round(45.926,0) 三,
	       round(45.926,-1) 四,round(45.926,-2) 五
	from dual;
```

	        一         二         三         四         五 
	---------- ---------- ---------- ---------- ---------- 
	     45.93       45.9         46         50          0 
-----------------------------------------------
#### 当前系统时间

```sql
	select sysdate from dual;
```

	SYSDATE       
	--------------
	09-2月 -15    
#### to_char

```sql
	select to_char(sysdate,'yyyy-mm-dd hh24:mi:ss') from dual;
```

	TO_CHAR(SYSDATE,'YY 
	------------------- 
	2015-02-09 14:26:47 
-----------------------------------------------
#### 昨天 今天 明天

```sql
	select (sysdate-1) 昨天,sysdate 今天,(sysdate+1) 明天
	from dual;
```

	昨天           今天           明天          
	-------------- -------------- --------------
	08-2月 -15     09-2月 -15     10-2月 -15  
-----------------------------------------------
#### 计算员工的工龄：天 星期 月 年

```sql
	select ename,hiredate,(sysdate-hiredate) 天,(sysdate-hiredate)/7 星期,
   	            (sysdate-hiredate)/30 月,(sysdate-hiredate)/365 年
	from emp;
```

-----------------------------------------------
#### 73个月后

```sql
	select add_months(sysdate,73) from dual;
```

	ADD_MONTHS(SYS    
	--------------    
	09-3月 -21
-----------------------------------------------

#### 最后一天

```sql
	select last_day(sysdate) from dual;
```

	LAST_DAY(SYSDA  
	--------------  
	28-2月 -15 
-----------------------------------------------

#### next_day

```sql
	select next_day(sysdate,'星期一') from dual;
```

	NEXT_DAY(SYSDA   
	--------------   
	16-2月 -15

><font color="red">next_day的应用：每个星期一自动备份数据</font>

>	1. 分布式数据库
>	2. 快照`snapshot`

-----------------------------------------------
#### 对日期四舍五入和截断
```sql
	select round(sysdate,'month'),round(sysdate,'year') from dual;
```

	ROUND(SYSDATE, ROUND(SYSDATE, 
	-------------- -------------- 
	01-2月 -15     01-1月 -15    
-----------------------------------------------

#### 转换函数

```sql
	select to_char(sysdate,'yyyy-mm-dd hh24:mi:ss') from dual;
```

	TO_CHAR(SYSDATE,'YY 
	------------------- 
	2015-02-09 14:52:54
-----------------------------------------------
#### 查询员工薪水：两位小数，千位符，货币代码

```sql
	select to_char(sal,'L9,999.99') from emp;
```

	TO_CHAR(SAL,'L9,999 
	------------------- 
	           ￥800.00 
	         ￥1,600.00 
	         ￥1,250.00 
	         ￥2,975.00 
-----------------------------------------------
#### 通用函数
nvl2(a,b,c)当a=null的时候，返回c；否则返回b

```sql
	select sal*12+nvl2(comm,comm,0) from emp;
```

	SAL*12+NVL2(COMM,COMM,0)  
	------------------------  
	                    9600  
	                   19500  
	                   15500  
	                   35700  
	                   16400  
	                   34200 
-----------------------------------------------
#### nullif(a,b) 当a=b时候，返回null，否则返回a

```sql
	select nullif('abc','abc') 值 from dual;
```

	值   
	---  

	
```sql
	select nullif('abcd','abc') 值 from dual;
```

	值   
	---- 
	abcd 
-----------------------------------------------
#### coalesce 从左到右找到第一个不为null的值

```sql
	select comm,sal,coalesce(comm,sal) "第一个不为null的值"
	from emp;
```

	      COMM        SAL 第一个不为null的值  
	---------- ---------- ------------------  
	                  800                800  
	       300       1600                300  
	       500       1250                500  
	                 2975               2975  
	      1400       1250               1400  
-----------------------------------------------

#### 条件表达式
给员工涨工资，总裁1000 经理800 其他400

```sql
	select ename,job,sal 涨前,
	       case job when 'PRESIDENT' then sal+1000
	                when 'MANAGER' then sal+800
	                else sal+400
	        end 涨后
	from emp;
```

	ENAME      JOB             涨前       涨后 
	---------- --------- ---------- ---------- 
	SMITH      CLERK            800       1200 
	ALLEN      SALESMAN        1600       2000 
	WARD       SALESMAN        1250       1650 
	JONES      MANAGER         2975       3775 
-----------------------------------------------
#### <font color="red"> decode条件表达式</font>

```sql
	select ename,job,sal 涨前,
	       decode(job,'PRESIDENT',sal+1000,
	                  'MANAGER',sal+800,
	                            sal+400) 涨后
	from emp;
```

	ENAME      JOB             涨前       涨后 
	---------- --------- ---------- ---------- 
	SMITH      CLERK            800       1200 
	ALLEN      SALESMAN        1600       2000 
	WARD       SALESMAN        1250       1650 
	JONES      MANAGER         2975       3775 
-----------------------------------------------

-----------------------------------------------

