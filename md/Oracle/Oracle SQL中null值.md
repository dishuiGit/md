#### 1.包含null的表达式都为null

>```sql
>--查询员工信息：员工号 姓名 月薪 年薪 奖金 年收入
>select empno,ename,sal,sal*12,comm,sal*12+comm
>from emp;
>```

<font color="red">奖金为空的,年收入为空!</font>

-----------------------------------------------

#### 2.null永远！=null

>```sql
>--查询奖金为null的员工
>select *
>from emp
>where comm=null;
>```
>
><font color="red">未选定行</font>

#### 2.1.<font color="green">nvl</font>:如果列为空,取0

>```sql
>select empno,ename,sal,sal*12,comm,sal*12+nvl(comm,0)
>from emp;
>```

-----------------------------------------------

#### 3.如果集合中含有null，不能使用not  in；但可以使用in

>```sql
>select *
>from emp
>where deptno not in (10,20,null)
>```
>
><font color="red">未选定行!</font>
>\----------------------------------------------------
>```sql
>select *
>from emp
>where deptno in (10,20,null)
>```
>
><font color="green">正确</font>
><font color="red">思考题：上面的原因是什么？</font>
-----------------------------------------------

#### 4. null的排序

>		Oracle 中，null最大

-----------------------------------------------

#### 5. null值 组函数自动滤空；可以嵌套滤空函数来屏蔽他的滤空功能

>```sql
>	select count(*),count(nvl(comm,0)) from emp;
>```

	  COUNT(*) COUNT(NVL(COMM,0))     
	---------- ------------------     
	        14                 14	    

\-----------------------------------------------

--求部门的平均工资

>	```sql
>		select deptno,avg(sal)
>		from emp
>		group by deptno;
>	```

	    DEPTNO   AVG(SAL)   
	---------- ----------   
	        30 1566.66667   
	        20       2175   
	        10 2916.66667
-----------------------------------------------
