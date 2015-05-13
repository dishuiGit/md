## Oracle 基本查询
>#### 1. 当前用户
```sql
show user;
```
-----------------------------------------
>#### 2. 当前用户下的表
```sql
    select * from tab
    ```
-----------------------------------------
> #### 3. 员工表结构
```sql
desc emp;
```
-----------------------------------------
>#### 4. 去掉重复记录
```sql
--distinct 
select distinct deptno from emp;
```
-----------------------------------------
>#### 5. 表:伪表
```sql
--dual 
select 'hello'||'  World' 字符串 from dual;
```
>输出
>>字符串 
>>\------------  
>>hello  World 

## 过滤和排序

>
```sql
--查看10 号部门的员工
select *
from emp
where deptno=10;
```
---------------------------------------------
>
```sql
--字符串大小写敏感
--查询名叫KING的员工
select *
from emp
where ename='KING';
```
---------------------------------------------

>```sql
>--日期格式敏感
>--查询入职日期是17-11月-81的员工
>select *
>from emp
>where hiredate='17-11月-81';
>```

>结果
>>
\--------------------
EMPNO ENAME    JOB              MGR HIREDATE         SAL       COMM     DEPTNO
\---------- -------- --------- ---------- -------------- ----- ---------- ----------
7839 KING     PRESIDENT            17-11月-81      5000                    10
---------------------------------------------
>--between and
>__查询薪水1000~2000之间的员工__
>```sql
>select *
>from emp
>where sal between 1000 and 2000;
>```
---------------------------------------------
>__查询部门号是10和20的员工__
>```sql
>select *
>from emp
>where deptno in (10,20);
>```
---------------------------------------------
>__查询部门号不是10和20的员工__
>```sql
>select *
>from emp
>where deptno not in (10,20)
>```
---------------------------------------------
>--模糊查询 % _
>__查询名字以S打头员工__
>```sql
>select *
>from emp
>where ename like 'S%';
>```
---------------------------------------------
>__查询名字是4个字的员工__
>```sql
>select *
>from emp
>where ename like '____';
>```
---------------------------------------------
>__查询名字中含有下划线的员工__
>```sql
>select *
>from emp
>where ename like '%\_%' escape '\'
>```
---------------------------------------------
>__按奖金降序排序,null值排在最后__
>```sql
>select * 
>from emp 
>order by comm desc
>nulls last
>```
---------------------------------------------
>__order by后面 + 列，表达式，别名，序号__
>```sql
>select empno,ename,sal,sal*12
>from emp
>order by sal*12 desc;
>```
### <font color="red">小结：</font>
+ __order by__ 作用于后面所有的列，先按照第一个排序，再按照第二个列排序，以此类推
+ __desc__ 只作用于离他最近的一列





