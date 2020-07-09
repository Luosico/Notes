# MySQL

### 连接查询

- **where语句**

  - 先计算笛卡尔积，列出所有可能，严重影响性能，避免使用

  ```mysql
    	select name from user1,user2 where user1.id = user2.id;
  ```

​    

- **join语句**

  - inner join	等值连接 ：获取两个表中字段匹配关系的记录
  - left     join   左连接：获取左表所有记录，即使右表没有对应匹配的记录
  - right  join   右连接：获取右表所有记录，即使左表没有对应匹配的记录

  ```mysql
  	select name from user1 inner join user2 on user1.id = user2.id;
  ```



### union操作符

​	**用于连接两个以上的select语句的结果组合到一个结果集中**

​	**union[distinct/all]    默认使用distinct**

```mysql
select 查询1   union[distinct/all]   select查询2   
```



### Limit

**常用于分页**

```
select * from user limit [offset];

limit 5      等价于 limit 0,5
limit 5,10   查询6-10行
limit 5,-1   从第6行开始直到末尾
limit 5,10   从第11行开始，返回5行
```





### 常用函数

#### 1、聚集函数

- **count** ：指定字段的数据的行数
- **max**：最大值
- **min**：最小值
- **sum**：求和
- **avg**：平均值



#### 2、处理字符串的函数

- **concact(str1,str2,str3,...)**：拼接字符串 
- **strcmp(str1,str2)**：比较字符串大小
- **length(str)**：获取字符串的**字节**数
- **char_length(str)**：获取字符串的**字符**数
- **upper(x)、ucase(x)/lower(x)、lcase(x)**：字母的大小写转换



#### 3、处理数值的函数

- **abs**：求绝对值
- **ceil**：向上取整
- **floor**：向下取整
- **mod**：取模
- **rand**：产生 0 到 1 之间的随机数
- **round（x, y）**：四舍五入，x 为要处理的数，y 为要保留的小数位数
- **truncate(x, D)**：将一个数字（x）截断为指定的小数位数（D），若 D 为负，则 TRUNCATE(x, D) 函数使小数点左边的 D 位变为 0，若 D 为 0，则返回值没有小数点



#### 4、处理事件日期的函数

- **curdate、current_date**：获取**当前日期**，形式为：2020-07-06
- **curtime、current_time**：获取**当前时间**，形式为：14:25:29
- **now，sysdate**：获取**当前日期和时间**，形式为：2020-07-06 14:29:20, now是语句开始时的时间，sysdate是实时时间
- **month(date),monthname(date)**：从日期中选择出月份数
- **week(date)**：从日期中选择出周数
- **year(date)**：从日期中选择出年数
- **hour(time)**：从时间中选择出小时数
- **minute(time)**：从时间中选择出分钟数
- **weekday(date),dayname(date)**：从时间中选择出今天是周几



### 存储过程

​	**存储过程就是数据库SQL语言层面上的代码封装与复用，经第一次编译后，都不需要再次编译**

#### 优点

- 存储过程可封装，并隐藏复杂的商业逻辑
- 存储过程可以回传值，并可以接受参数
- 存储过程可以用在数据检验，强制实行商业逻辑等
- 执行的速度相对快一些
- 减少网络之间的数据传输，节省开销  

#### 缺点

- 存储过程无法使用 SELECT 指令来运行，因为它是子程序，与查看表，数据表或用户定义函数不同
- 复杂的业务逻辑，用存储过程还是很吃力的
- 对于开发和调试都很不方便
- 可移植性太差了



#### 创建存储过程

```SQL
CREATE
    [DEFINER = { user | CURRENT_USER }]
　PROCEDURE procedure_name ([proc_parameter[,...]])
    [characteristic ...] routine_body
 
proc_parameter:
    [ IN | OUT | INOUT ] param_name type
 
characteristic:
    COMMENT 'string'
  | LANGUAGE SQL
  | [NOT] DETERMINISTIC
  | { CONTAINS SQL | NO SQL | READS SQL DATA | MODIFIES SQL DATA }
  | SQL SECURITY { DEFINER | INVOKER }
 
routine_body:
　　Valid SQL routine statement
 
[begin_label:] BEGIN
　　[statement_list]
　　　　……
END [end_label]
```



**最简单的存储过程**

```
#创建存储过程
create procedure test()
begin
	select * from user;
end;

#调用存储过程
call test();
```



**使用变量**

- 变量的声明使用 **declare**,一句declare只声明一个变量，变量必须先声明后使用
- 变量具有数据类型和长度，与 MySQL 的 SQL 数据类型保持一致，因此甚至还能制定默认值、字符集和排序规则等
- 变量可以通过 **set**来赋值，也可以通过 **select into**的方式赋值
- 变量需要**返回**，可以使用select语句，如：select  变量名

```sql
create procedure test2()
begin
  -- 使用 declare语句声明一个变量
  declare username varchar(32) default '';
  -- 使用set语句给变量赋值
  set username='xiaoxiao';
  -- 将users表中id=1的名称赋值给username
  select name into username from users where id=1;
  -- 返回变量
  select username;
end;
```



#### 变量作用域

- 存储过程中变量是有作用域的，作用范围在begin和end块之间，end结束变量的作用范围即结束
- 需要多个块之间传值，可以使用全局变量，即放在所有代码块之前
- 传参变量是全局的，可以在多个块之间起作用

```SQL
CREATE DEFINER=`root`@`localhost` PROCEDURE `test2`()
begin
	begin
		declare n1 int;
		declare n2 int;
		
		select count(*) into n1 from class;
		select count(*) into n2 from course;
		
		select n1,n2;
	end;
	
	begin
		declare n3 int;
		declare n4 int;
		
		select count(*) into n3	from score;
		select count(*) into n3 from student;
		
		select n3,n4;
	end;
end
```

这里显示两个结果

![Ue58vd.png](https://s1.ax1x.com/2020/07/09/Ue58vd.png)
![UeoPl4.png](https://s1.ax1x.com/2020/07/09/UeoPl4.png)



#### 存储过程参数（ in、out、inout ）

```SQL
create procedure 名称([IN|OUT|INOUT] 参数名 参数数据类型 )
begin
.........
end
```

##### 1、in 输入参数

 调用者向存储过程传入值（传入值可以是字面量或变量）

```SQL
-- 创建存储过程
create PROCEDURE `test`(in id int)
begin
	--	返回变量
	select id;
	-- 修改变量值
	set id=5;
	select id;
end

-- 调用存储过程
call test(1)
-- 返回值是 1

set @idd=2;
call test(@idd);
-- 返回值仍然是传入的值
-- 因为前者为局部变量，后者为全局变量
select @idd;
```

![UmJuex.png](https://s1.ax1x.com/2020/07/09/UmJuex.png)

![UmGbo8.png](https://s1.ax1x.com/2020/07/09/UmGbo8.png)

##### 2、out 输出参数

表示存储过程向调用者传出值（可以返回多个值，传出值只能是变量），不能用于传入参数值，中间改变其值会被改变

```sql
CREATE PROCEDURE `test`(out id int)
begin
	select id;
	set id=5;
	select id;
end

-- 调用存储过程
set @idd=2; -- 变量可不同名
call test(@idd); -- 不传入变量会报错
select @idd;
```

​	**因为out是向调用者输出参数，不接收输入的参数，所以存储过程里的id为null，最终的结果改变了@idd**

![Um3ZQI.png](https://s1.ax1x.com/2020/07/09/Um3ZQI.png)

##### 3、inout

```sql
CREATE PROCEDURE `test2`(inout id int)
begin
	select id;
	set id=5;
	select id;
end

-- 调用存储过程
set @idd=2; -- 变量可不同名
call test(@idd); -- 不传入变量会报错
select @idd;
```

![Um8y4g.png](https://s1.ax1x.com/2020/07/09/Um8y4g.png)