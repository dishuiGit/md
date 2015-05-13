### 外连接
--按部门统计员工人数：部门号 部门名称 人数

```sql 
	select d.deptno 部门号,d.dname 部门名称,count(e.empno) 人数
	from emp e,dept d
	where e.deptno=d.deptno
	group by d.deptno,d.dname;
```

	    部门号 部门名称             人数 
	---------- -------------- ---------- 
	        10 ACCOUNTING              3 
	        20 RESEARCH                5 
	        30 SALES                   6 

------------------------------------------------------------

#### 希望：对于某些不成立的记录，任然希望包含在最后的结果中

+ <font color="blue">*左外连接：*</font>

	当`where e.deptno=d.deptno`不成立的时候，等号左边的表任然被包含
	     **写法**：`where e.deptno=d.deptno(+)`

+ <font color="blue">*右外连接：*</font>

	当`where e.deptno=d.deptno`不成立的时候，等号右边的表任然被包含
	     **写法**：`where e.deptno(+)=d.deptno`

```sql
	select d.deptno 部门号,d.dname 部门名称,count(e.empno) 人数
	from emp e,dept d
	where e.deptno(+)=d.deptno
	group by d.deptno,d.dname;
```

	    部门号 部门名称             人数
	---------- -------------- ----------
	        10 ACCOUNTING              3
	        40 OPERATIONS              0
	        20 RESEARCH                5
	        30 SALES                   6        

------------------------------------------------------------	        

### 层次查询
*自连接不适合操作大表*

```sql
	select level,empno,ename,mgr
	from emp
	connect by prior empno=mgr
	start with mgr is null
	order by 1;
```

	     LEVEL      EMPNO ENAME             MGR    
	---------- ---------- ---------- ----------    
	         1       7839 KING                     
	         2       7566 JONES            7839    
	         2       7698 BLAKE            7839    
	         2       7782 CLARK            7839    
	         3       7902 FORD             7566    
	         3       7521 WARD             7698    
	         3       7900 JAMES            7698    
	         3       7934 MILLER           7782    
	         3       7499 ALLEN            7698    
	         3       7788 SCOTT            7566    
	         3       7654 MARTIN           7698    



