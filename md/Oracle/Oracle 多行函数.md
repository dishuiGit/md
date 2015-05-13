#### 工资总额

>```sql
>	select sum(sal) from emp;
>```

	SUM(SAL)   
	---------- 
	     29025 

-------------------------------------------------
#### 人数

```sql
	select count(*) from emp;
```

	  COUNT(*) 
	---------- 
	        14 

-------------------------------------------------
#### 平均工资

>```sql
>	select sum(sal)/count(*) 一,avg(sal) 二 from emp;
>```

	        一         二    
	---------- ----------    
	2073.21429 2073.21429 

-------------------------------------------------
#### 平均奖金

>```sql
>	select sum(comm)/count(*) 一,sum(comm)/count(comm) 二,avg(comm) 三
>	from emp;
>```

	        一         二         三     
	---------- ---------- ----------     
	157.142857        550        550  

-------------------------------------------------

>```sql
>	select count(*),count(comm) from emp;
>```

	  COUNT(*) COUNT(COMM)    
	---------- -----------    
	        14           4  

-------------------------------------------------

#### 多个列的分组

>```sql
>	select deptno,job,sum(sal)
>	from emp
>	group by deptno,job
>	order by 1;
>```

	    DEPTNO JOB         SUM(SAL)     
	---------- --------- ----------     
	        10 CLERK           1300     
	        10 MANAGER         2450     
	        10 PRESIDENT       5000     
	        20 ANALYST         6000                 

-------------------------------------------------

#### 过滤分组数据
--查询平均工资大于2000的部门

>```sql
>	select deptno,avg(sal)
>	from emp
>	group by deptno
>	having avg(sal)>2000;
>```

	    DEPTNO   AVG(SAL)     
	---------- ----------     
	        20       2175     
	        10 2916.66667     

-------------------------------------------------

### <font color="red">where和having最大的区别：where后面不能使用组函数</font>
--查询10号部门的平均工资

>```sql
>	select deptno,avg(sal)
>	from emp
>	group by deptno
>	having deptno=10;
>```

	    DEPTNO   AVG(SAL)   
	---------- ----------   
	        10 2916.66667   

-------------------------------------------------

### <font color="red">group by的增强</font>

	select deptno,job,sum(sal) from emp group by deptno,job
	+
	select deptno,sum(sal) from emp group by deptno
	+
	select sum(sal) from emp
	==========================
	select deptno,job,sum(sal) from emp group by rollup(deptno,job);
	
	group by rollup(a,b)
	=
	group by a,b
	+
	group by a
	+
	group by null

\----------------------------------------------

>```sql
>	select deptno,job,sum(sal) from emp group by rollup(deptno,job);
>```

	    DEPTNO JOB         SUM(SAL)    
	---------- --------- ----------    
	        10 CLERK           1300    
	        10 MANAGER         2450    
	        10 PRESIDENT       5000    
	        10                 8750    
	        20 CLERK           1900    
	        20 ANALYST         6000    
	        20 MANAGER         2975    
	        20                10875    
	        30 CLERK            950    
	        30 MANAGER         2850    
	        30 SALESMAN        5600                       

-------------------------------------------------

-------------------------------------------------

-------------------------------------------------