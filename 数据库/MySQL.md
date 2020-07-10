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



#### 局部变量

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

#### 用户变量

​	**在调用存储过程前定义的变量，一般以 @ 开头**

​	两种方式设置用户变量

```sql
-- 第一种
set @param='hello';

-- 第二种
select 'hello' into @param;
```



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

#### 游标

​	**游标是保存查询结果的临时区域**

```sql
-- 使用游标，把users表中 id为偶数的记录逐一更新用户名

create procedure test11()
    begin
        declare stopflag int default 0;
        declare username VARCHAR(32);
        -- 创建一个游标变量，declare 变量名 cursor ...
        declare username_cur cursor for select name from users where id%2=0;
        -- 游标是保存查询结果的临时区域
        -- 游标变量username_cur保存了查询的临时结果，实际上就是结果集
        -- 当游标变量中保存的结果都查询一遍(遍历),到达结尾，将变量stopflag设置为1，用于循环中判断是否结束
        declare continue handler for not found set stopflag=1;
 
        open username_cur; -- 打开游标
        fetch username_cur into username; -- 游标向前走一步，取出一条记录放到变量username中
        while(stopflag=0) do -- 如果游标还没有结尾，就继续
            begin 
                -- 在用户名前门拼接 '_cur' 字符串
                update users set name=CONCAT(username,'_cur') where name=username;
                fetch username_cur into username;
            end;
        end while; -- 结束循环
        close username_cur; -- 关闭游标
    end;
```



### 函数

#### 1、函数与存储过程的区别

​	存储过程和函数是事先经过编译并存储在数据库中的**一段SQL语句的集合**，减少数据在数据库和应用服务之间的传输，对于提高数据处理的效率是有好处的

- **函数必须有返回值**，而**存储过程没有**，存储过程的参数可以使用**IN**、**OUT**、**INOUT**类型，而函数的参数只能是**IN**类型的。如果有函数从其他类型的数据库迁移到MySQL，可能需要将函数改造成存储过程。、

```sql
create function getusername(userid int) returns varchar(32)
    reads sql data  -- 从数据库中读取数据，但不修改数据
    begin
        declare username varchar(32) default '';
        select name into username from users where id=userid;
        return username;
    end;
```



### 自增字段 auto_increment

​	**使用 auto_increment的列必须使用了索引，新插入的值每次都会从最大值开始增加，除非显示指定**

​	**InnoDB引擎的自增值，在MySQL5.7及之前的版本，自增值保存在内存里，并没有持久化。每次重启后，第一次打开表的时候，都会去找自增值的最大值max(id)，然后将max(id)+步长作为这个表当前的自增值**

#### 1、插入

- 如果把一个NULL插入到一个AUTO_INCREMENT数据列里去，MySQL将自动生成下一个序列编号。编号从1开始，并1为基数递增
- 当插入记录时，没有为AUTO_INCREMENT明确指定值，则等同插入NULL值
- 当指定一个明确的值给自增列的时候
    - 大于自增列的最大值：能插入成功，之后从这个值开始增加
    - 小于自增列的最大值，且该值未被使用：能插入成功，之后从最大值开始增加

#### 2、更新

​	只要设置的新值还未被使用，就能插入成功，之后从最大值开始增加

#### 3、自定义偏移值和步长

- auto_increment_offset   自增的起始值
- auto_increment_increment   自增的步长

#### 4、自增锁的优化

MySQL5.0版本的时候，自增锁的范围是**语句**级别。也就是说，如果一个语句申请了一个表自增锁，这个锁会等语句执行结束以后才释放

MySQL5.1.22版本引入了一个新策略，新增参数innodb_autoinc_lock_mode，默认值是1

- 这个参数设置为0，表示采用之前MySQL5.0版本的策略，即语句执行结束后才释放锁
- 这个参数设置为1
    - 普通insert语句，自增锁在申请之后就**马上释放**
    - 类似insert … select这样的批量插入数据的语句，自增锁还是要等语句结束后才被释放
- .这个参数设置为2，所有的申请自增主键的动作都是申请后就释放锁

**对于批量插入数据的语句，MySQL有一个批量申请自增id的策略：**

1.语句执行过程中，第一次申请自增id，会分配1个

2.1个用完以后，这个语句第二次申请自增id，会分配2个

3.2个用完以后，还是这个语句，第三次申请自增id，会分配4个

4.依次类推，同一个语句去申请自增id，每次申请到的自增id个数都是上一次的两倍

#### 5、自增值用完，达到最大值

​		数据库表的自增 ID 达到上限之后，再申请时它的值就不会在改变了，继续插入数据时会导致报主键冲突错误。因此在设计数据表时，尽量根据业务需求来选择合适的字段类型

​		一般用int，若用完单表得上亿条数据，实际环境单表不会让其这么大，会使用分库分表



### count(*)/count(常数)/count(field)

**count(*)和count(常数)**表示的是直接查询符合条件的数据库表的行数，包括**NULL**

**count(field)**是查询符合条件的**不为NUL**L的行数



对于**count(***)和**count(1)**：官方文档指出对其优化是一样的，所有两者没什么区别

优化方式是：优先选择最小的非聚簇索引来扫表，前提是查询语句不含where和group by



### 控制语句

#### 1、条件语句

##### if-else-then

```sql
-- 基本结构
if() then
	...
else
	...
end if;

-- 嵌套结构
if() then
	...
else if() then
		...
	else
		...
end if;
```

##### case

- 当colume 与condition 条件相等时结果为result

```sql
case colume 
    when condition then result
    when condition then result
    when condition then result
else result
end
```

- 当满足某一条件时，执行某一result

```sql
case
    when condition then result
    when condition then result
    when condition then result
else result
end
```

- 当满足某一条件时，执行某一result,把该结果赋值到new_column_name 字段中

```sql
case  
    when condition then result
    when condition then result
    when condition then result
else result
end new_column_name
```



#### 2、循环语句

##### while

```sql
-- 不符合循环条件跳出循环
while(表达式) do 
   ......  
end while;
```

##### repeat

```sql
-- 符合循环条件跳出循环
repeat
   ...
until 循环条件  
end repeat;
```

##### loop

```sql
loop_name:loop -- 循环开始
	if i>a then 
		leave loop_name;  -- 判断条件成立则结束循环  好比java中的 break
    end if;
        set sum=sum+i;
        set i=i+1;
end loop;  -- 循环结束
```



### 触发器

 当对表进行 insert、update、delete 时触发操作

```sql
create trigger trigger_name after insert on users
    for each row 
    begin 
        insert into oplog(userid,username,action,optime)
        values(NEW.id,NEW.name,'insert',now());
    end;
```

