[TOC]

## 查询工资比SCOTT高的员工信息
#### 1. SCOTT的工资

>```sql
>	select sal from emp where ename='SCOTT';
>```
>		
>		       SAL   
>		----------   
>		      3000   

----------------------------------------------------------

#### 2. 比3000高的员工

>```sql
>	select * from emp where sal > 3000;
>```
>
> 	    EMPNO ENAME      JOB              MGR HIREDATE              SAL       COMM     DEPTNO    
> 	     7839 KING       PRESIDENT            17-11月-81           5000                    10 

----------------------------------------------------------

#### 3.子查询所要解决的问题：不能一步求解

>```sql
>select *
>from emp
>where sal > (select sal
>             from emp
>             where ename='SCOTT');
>```
>
>		     EMPNO ENAME      JOB              MGR HIREDATE              SAL       COMM     DEPTNO   
>		---------- ---------- --------- ---------- -------------- ---------- ---------- ----------
>		      7839 KING       PRESIDENT            17-11月-81           5000                    10         

----------------------------------------------------------

# <font color="red">注意的问题：</font>

>1. **括号**
>2. **合理的书写风格**
>3. **可以在主查询的`where` `select` `having` `from`后面放置子查询**
>4. **不可以在group by语句后面放置子查询**
>5. **强调`from`后面的子查询**
>6. **主查询和子查询可以不是同一张表；只要子查询返回的结果 主查询可以使用 ，即可**
>7. **一般不在子查询中排序；但在`Top-N`分析问题中，必须对子查询排序**
>8. **一般先执行子查询，再执行主查询；但相关子查询例外**
>9. **单行子查询只能使用单行操作符；多行子查询只能使用多行操作符**
>10. **子查询中的null**

----------------------------------------------------------

#### 3. 可以在主查询的`where ` `select` `having` `from`后面放置子查询

```sql
	select empno,ename,sal,(select job from emp where empno=7839) 第四列
	from emp;
```

	     EMPNO ENAME             SAL 第四列    
	---------- ---------- ---------- --------- 
	      7369 SMITH             800 PRESIDENT 
	      7499 ALLEN            1600 PRESIDENT 
	      7521 WARD             1250 PRESIDENT 
	      7566 JONES            2975 PRESIDENT                          

----------------------------------------------------------

#### 5. 强调from后面的子查询

**查询员工信息：员工号 姓名 月薪**

```sql
	select *
	from (select empno,ename,sal from emp);
```

	     EMPNO ENAME             SAL      
	---------- ---------- ----------      
	      7369 SMITH             800      
	      7499 ALLEN            1600      
	      7521 WARD             1250      
	      7566 JONES            2975      
	      7654 MARTIN           1250      
	      7698 BLAKE            2850      
	      7782 CLARK            2450      
	      7788 SCOTT            3000       

----------------------------------------------------------

#### 6. 主查询和子查询可以不是同一张表；只要子查询返回的结果 主查询可以使用 ，即可
**查询部门名称是SALES的员工**

```sql 
	select *
	from emp
	where deptno=(select deptno
	              from dept
	              where dname='SALES');
```

	     EMPNO ENAME      JOB              MGR HIREDATE              SAL       COMM     DEPTNO  
	---------- ---------- --------- ---------- -------------- ---------- ---------- ----------  
	      7499 ALLEN      SALESMAN        7698 20-2月 -81           1600        300         30  
	      7521 WARD       SALESMAN        7698 22-2月 -81           1250        500         30  
	      7654 MARTIN     SALESMAN        7698 28-9月 -81           1250       1400         30  
	      7698 BLAKE      MANAGER         7839 01-5月 -81           2850                    30  
	      7844 TURNER     SALESMAN        7698 08-9月 -81           1500          0         30  
	      7900 JAMES      CLERK           7698 03-12月-81            950                    30           

----------------------------------------------------------

### **<font color="red">in</font>** 在集合中

**查询部门名称是`SALES`和`ACCOUNTING`的员工**

```sql
	select * --子查询
	from emp
	where deptno in (select deptno from dept where dname='SALES' or dname='ACCOUNTING');
```

```sql
	select e.*
	from emp e,dept d --多表查询(SQL优化原则,尽量用多表查询!)
	where e.deptno=d.deptno and (d.dname='SALES' or d.dname='ACCOUNTING');
```

	     EMPNO ENAME      JOB              MGR HIREDATE              SAL       COMM     DEPTNO 
	---------- ---------- --------- ---------- -------------- ---------- ---------- ---------- 
	      7499 ALLEN      SALESMAN        7698 20-2月 -81           1600        300         30 
	      7521 WARD       SALESMAN        7698 22-2月 -81           1250        500         30 
	      7654 MARTIN     SALESMAN        7698 28-9月 -81           1250       1400         30 
	      7698 BLAKE      MANAGER         7839 01-5月 -81           2850                    30 
	      7782 CLARK      MANAGER         7839 09-6月 -81           2450                    10 
	      7839 KING       PRESIDENT            17-11月-81           5000                    10 
	      7844 TURNER     SALESMAN        7698 08-9月 -81           1500          0         30 
	      7900 JAMES      CLERK           7698 03-12月-81            950                    30 
	      7934 MILLER     CLERK           7782 23-1月 -82           1300                    10           

----------------------------------------------------------

#### **<font color="red">any</font>**: 和集合中的任意一个值比较
**查询工资比30号部门任意一个员工高的员工信息**

```sql
	select *
	from emp
	where sal > any (select sal from emp where deptno=30);
```

	     EMPNO ENAME      JOB              MGR HIREDATE              SAL       COMM     DEPTNO  
	---------- ---------- --------- ---------- -------------- ---------- ---------- ----------  
	      7839 KING       PRESIDENT            17-11月-81           5000                    10  
	      7902 FORD       ANALYST         7566 03-12月-81           3000                    20  
	      7788 SCOTT      ANALYST         7566 19-4月 -87           3000                    20  
	      7566 JONES      MANAGER         7839 02-4月 -81           2975                    20  
	      7698 BLAKE      MANAGER         7839 01-5月 -81           2850                    30  
	      7782 CLARK      MANAGER         7839 09-6月 -81           2450                    10  
	      7499 ALLEN      SALESMAN        7698 20-2月 -81           1600        300         30  
	      7844 TURNER     SALESMAN        7698 08-9月 -81           1500          0         30  
	      7934 MILLER     CLERK           7782 23-1月 -82           1300                    10  
	      7521 WARD       SALESMAN        7698 22-2月 -81           1250        500         30  
	      7654 MARTIN     SALESMAN        7698 28-9月 -81           1250       1400         30  
	      7876 ADAMS      CLERK           7788 23-5月 -87           1100                    20  


----------------------------------------------------------

### 多行子查询中的null

`not in (10,20,null)`

**查询不是老板的员工**

```sql
	select *
	from emp
	where empno not in (select mgr from emp);
```

*<font color="red">未选定行</font>*

```sql
	select *
	from emp
	where empno not in (select mgr from emp where mgr is not null);
```

	       EMPNO ENAME      JOB              MGR HIREDATE              SAL       COMM     DEPTNO   
	---------- ---------- --------- ---------- -------------- ---------- ---------- ----------     
	      7844 TURNER     SALESMAN        7698 08-9月 -81           1500          0         30     
	      7521 WARD       SALESMAN        7698 22-2月 -81           1250        500         30     
	      7654 MARTIN     SALESMAN        7698 28-9月 -81           1250       1400         30     
	      7499 ALLEN      SALESMAN        7698 20-2月 -81           1600        300         30     
	      7934 MILLER     CLERK           7782 23-1月 -82           1300                    10     
	      7369 SMITH      CLERK           7902 17-12月-80            800                    20     
	      7876 ADAMS      CLERK           7788 23-5月 -87           1100                    20     
	      7900 JAMES      CLERK           7698 03-12月-81            950                    30  




