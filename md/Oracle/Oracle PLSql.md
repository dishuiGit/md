### PLSql **Hello World**

```sql
	declare
	begin
		dbms_output.put_line('Hello World');	
	end;
	/
```

------------------------------------------------------

### PLSql 变量和常量说明

```sql
	var1	char(15);
	--说明变量名,数据类型和长度后用分号结束说明语句
	married	boolean :=true;
	psal	number(7,2);
	my_name	emp.ename%type;
	--引用型变量,即my_name的类型与emp表中ename列的类型一样
	emp_rec	emp%rowtype;
	--记录型变量
```

------------------------------------------------------

### if语句

```sql
	--判断用户从键盘上输入的数字
	set serveroutput on
	
	--接收键盘输入
	--num: 地址值，在该地址上保存了输入的值
	accept num prompt '请输入一个数字';
	
	declare
	  --定义变量保存输入的数字
	  pnum number := &num;
	begin
	  if pnum = 0 then dbms_output.put_line('您输入的是0');
	    elsif pnum = 1 then dbms_output.put_line('您输入的是1');
	    elsif pnum = 2 then dbms_output.put_line('您输入的是2');
	    else dbms_output.put_line('其他数字');
	  end if;
	end;
	/
```

------------------------------------------------------

### 光标

```sql
	set serveroutput on
	declare
	  --定义光标
	  cursor cemp is select ename,sal from emp;
	  pname emp.ename%type;
	  psal  emp.sal%type;
	  
	begin
	  --打开光标
	  open cemp;
	  --循环
	  loop
	    --取出一条记录,存入pename,psal
	    fetch cemp into pname,psal;
	    --exit when 没有取到记录;
	    exit when cemp%notfound;
	    
	    dbms_output.put_line(pname||'的薪水是'||psal);
	  end loop;
	  --关闭光标
	  close cemp;
	end;
```

------------------------------------------------------

### 查询并打印7839的姓名和薪水

```sql
	set serveroutput on
	declare
	  --定义记录型变量：代表一行
	  emp_rec emp%rowtype;
	begin
	  select * into emp_rec from emp where empno=7839;
	  dbms_output.put_line(emp_rec.ename||'的薪水是'||emp_rec.sal);
	end;
	/
```

------------------------------------------------------

### 查询某个部门的员工姓名

```sql
set serveroutput on

declare
  --定义光标 
  cursor cemp(dno number) is select ename from emp where deptno=dno;
  pename emp.ename%type;
begin
  open cemp(20);
  loop
    fetch cemp into pename;
    exit when cemp%notfound;

    dbms_output.put_line(pename);

  end loop;
  close cemp;
end;
/
```

------------------------------------------------------

------------------------------------------------------