>#### 显示/设置行宽
```sql
show linesize
--linesize 80

set linesize 120
```
------------------------------------------------------

>#### 设置列宽
```sql
col ename for a8
col sal for 9999
```
------------------------------------------------------

>#### 查看日期格式
>```sql
>select * from v$nls_parameters;
>```
>#### 修改日期格式
>```sql
>alter session set NLS_DATE_FORMAT='yyyy-mm-dd';
>```

------------------------------------------------------


