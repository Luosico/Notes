# MySQL

## 连接查询

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



## union操作符

​	**用于连接两个以上的select语句的结果组合到一个结果集中**

​	**union[distinct/all]    默认使用distinct**

```mysql
select 查询1   union[distinct/all]   select查询2   
```



## Limit

**常用于分页**

```
select * from user limit [offset];

limit 5      等价于 limit 0,5
limit 5,10   查询6-10行
limit 5,-1   从第6行开始直到末尾
limit 5,10   从第11行开始，返回5行
```





## 1、常用函数

#### 1.1、聚集函数

- **count** ：指定字段的数据的行数
- **max**：最大值
- **min**：最小值
- **sum**：求和
- **avg**：平均值



#### 1.2、处理字符串的函数

- **concact(str1,str2,str3,...)**：拼接字符串 
- **strcmp(str1,str2)**：比较字符串大小
- **length(str)**：获取字符串的**字节**数
- **char_length(str)**：获取字符串的**字符**数
- **upper(x)、ucase(x)/lower(x)、lcase(x)**：字母的大小写转换



#### 1.3、处理数值的函数

- **abs**：求绝对值
- **ceil**：向上取整
- **floor**：向下取整
- **mod**：取模
- **rand**：产生 0 到 1 之间的随机数
- **round（x, y）**：四舍五入，x 为要处理的数，y 为要保留的小数位数
- **truncate(x, D)**：将一个数字（x）截断为指定的小数位数（D），若 D 为负，则 TRUNCATE(x, D) 函数使小数点左边的 D 位变为 0，若 D 为 0，则返回值没有小数点



#### 1.4、处理事件日期的函数

- **curdate、current_date**：获取**当前日期**，形式为：2020-07-06
- **curtime、current_time**：获取**当前时间**，形式为：14:25:29
- **now，sysdate**：获取**当前日期和时间**，形式为：2020-07-06 14:29:20, now是语句开始时的时间，sysdate是实时时间
- **month(date),monthname(date)**：从日期中选择出月份数
- **week(date)**：从日期中选择出周数
- **year(date)**：从日期中选择出年数
- **hour(time)**：从时间中选择出小时数
- **minute(time)**：从时间中选择出分钟数
- **weekday(date),dayname(date)**：从时间中选择出今天是周几

#### 1.5、group by

![w3fOc6.png](https://s1.ax1x.com/2020/09/09/w3fOc6.png)

​	如果直接使用有重复值的列，如 `select * from t group by name`，将会报错；**因为关系数据库就是基于关系的，单元格中是不允许有多个值的**，此时必须使用**聚合函数**来聚集数据，将多行聚集称为一行

​	**当使用 order by 的时候，必须放在 group by 的后面**



## 2、存储过程

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
        whi	`le(stopflag=0) do -- 如果游标还没有结尾，就继续
            begin 
                -- 在用户名前门拼接 '_cur' 字符串
                update users set name=CONCAT(username,'_cur') where name=username;
                fetch username_cur into username;
            end;
        end while; -- 结束循环
        close username_cur; -- 关闭游标
    end;
```



## 3、函数

#### 3.1、函数与存储过程的区别

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



## 4、自增字段 auto_increment

​	**使用 auto_increment的列必须使用了索引，新插入的值每次都会从最大值开始增加，除非显示指定**

​	**InnoDB引擎的自增值，在MySQL5.7及之前的版本，自增值保存在内存里，并没有持久化。每次重启后，第一次打开表的时候，都会去找自增值的最大值max(id)，然后将max(id)+步长作为这个表当前的自增值**

#### 4.1、插入

- 如果把一个NULL插入到一个AUTO_INCREMENT数据列里去，MySQL将自动生成下一个序列编号。编号从1开始，并1为基数递增
- 当插入记录时，没有为AUTO_INCREMENT明确指定值，则等同插入NULL值
- 当指定一个明确的值给自增列的时候
    - 大于自增列的最大值：能插入成功，之后从这个值开始增加
    - 小于自增列的最大值，且该值未被使用：能插入成功，之后从最大值开始增加

#### 4.2、更新

​	只要设置的新值还未被使用，就能插入成功，之后从最大值开始增加

#### 4.3、自定义偏移值和步长

- auto_increment_offset   自增的起始值
- auto_increment_increment   自增的步长

#### 4.4、自增锁的优化

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

#### 4.5、自增值用完，达到最大值

​		数据库表的自增 ID 达到上限之后，再申请时它的值就不会在改变了，继续插入数据时会导致报主键冲突错误。因此在设计数据表时，尽量根据业务需求来选择合适的字段类型

​		一般用int，若用完单表得上亿条数据，实际环境单表不会让其这么大，会使用分库分表



## 5、count(*)/count(常数)/count(field)

**count(*)和count(常数)**表示的是直接查询符合条件的数据库表的行数，包括**NULL**

**count(field)**是查询符合条件的**不为NUL**L的行数



对于**count(***)和**count(1)**：官方文档指出对其优化是一样的，所有两者没什么区别

优化方式是：优先选择最小的非聚簇索引来扫表，前提是查询语句不含where和group by



## 6、控制语句

#### 6.1、条件语句

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



#### 6.2、循环语句

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



## 7、触发器

 当对表进行 insert、update、delete 时触发操作

```sql
create trigger trigger_name after insert on users
    for each row 
    begin 
        insert into oplog(userid,username,action,optime)
        values(NEW.id,NEW.name,'insert',now());
    end;
```

## 8、分区、分库、分表

#### 8.1、分区

​	逻辑上还是一张表，物理上分成不同的表，把一张表的数据分成N多个区块，这些区块可以在同一个磁盘上，也可以在不同的磁盘上

```SQL
PARTITION BY RANGE(YEAR(order_day)) (

    PARTITION p_2010 VALUES LESS THAN (2010),

    PARTITION p_2011 VALUES LESS THAN (2011),

    PARTITION p_2012 VALUES LESS THAN (2012),

PARTITION p_catchall VALUES LESS THAN MAXVALUE);
```

##### 使用场景

- 一张表的查询速度已经慢到影响使用的时候
- 表中的数据是分段的
- .对数据的操作往往只涉及一部分数据，而不是所有的数据

#### 8.2、分表

​	真正从一张表分成不同的小表

##### 使用场景

- 一张表的查询速度已经慢到影响使用的时候
- 当频繁插入或者联合查询时，速度变慢

#### 8.3、分库

​	分表能够解决单表数据量过大带来的查询效率下降的问题，但是，却无法给数据库的并发处理能力带来质的提升。面对高并发的读写访问，当数据库master服务器无法承载写操作压力时，不管如何扩展slave服务器，此时都没有意义了。因此，我们必须换一种思路，对数据库进行拆分，从而提高数据库写入能力，这就是所谓的分库。

与分表策略相似，分库可以采用通过一个关键字取模的方式，来对数据访问进行路由



## 9、锁

### 9.1、表级锁

​	Mysql中锁定 **粒度最大** 的一种锁，对当前操作的整张表加锁，实现简单 **，资源消耗也比较少，加锁快，不会出现死锁** 。其锁定粒度最大，触发锁冲突的概率最高，并发度最低，MyISAM和 InnoDB引擎都支持表级锁。

- 事务更新大表中的大部分数据直接使用表级锁效率更高；
- 事务比较复杂，使用行级索很可能引起死锁导致回滚。

### 9.2、行级锁

​	Mysql中锁定 **粒度最小** 的一种锁，只针对当前操作的行进行加锁。 **行级锁能大大减少数据库操作的冲突。其加锁粒度最小，并发度高，但加锁的开销也最大，加锁慢，会出现死锁。** InnoDB支持的行级锁，包括如下几种：

- **Record Lock:** 对索引项加锁，锁定符合条件的行。其他事务不能修改和删除加锁项；
- **Gap Lock:** 对索引项之间的“间隙”加锁，锁定记录的范围（对第一条记录前的间隙或最后一条将记录后的间隙加锁），不包含索引项本身。其他事务不能在锁范围内插入数据，这样就防止了别的事务新增幻影行。
- **Next-key Lock：** 锁定索引项本身和索引范围。即Record Lock和Gap Lock的结合。可解决幻读问题。

**相关知识点：**

1. innodb对于行的查询使用next-key lock
2. Next-locking keying为了解决Phantom Problem幻读问题
3. 当查询的索引含有唯一属性时，将next-key lock降级为record key
4. Gap锁设计的目的是为了阻止多个事务将记录插入到同一范围内，而这会导致幻读问题的产生
5. 有两种方式显式关闭gap锁：（除了外键约束和唯一性检查外，其余情况仅使用record lock） A. 将事务隔离级别设置为RC B. 将参数innodb_locks_unsafe_for_binlog设置为1



​    **InnoDB的行级锁是基于索引实现的，如果查询语句为命中任何索引，那么InnoDB会使用表级锁.** 此外，InnoDB的行级锁是针对索引加的锁，不针对数据记录，因此即使访问不同行的记录，如果使用了相同的索引键仍然会出现锁冲突

​	**使用锁的时候，如果表没有定义任何索引，那么InnoDB会创建一个隐藏的聚簇索引并使用这个索引来加记录锁**

 

## 10、大表优化

当MySQL单表记录数过大时，数据库的CRUD性能会明显下降，一些常见的优化措施如下：

#### 1. 限定数据的范围

务必禁止不带任何限制数据范围条件的查询语句。比如：我们当用户在查询订单历史的时候，我们可以控制在一个月的范围内；

#### 2. 读/写分离

经典的数据库拆分方案，主库负责写，从库负责读；

#### 3. 垂直分区

**根据数据库里面数据表的相关性进行拆分。** 例如，用户表中既有用户的登录信息又有用户的基本信息，可以将用户表拆分成两个单独的表，甚至放到单独的库做分库。

**简单来说垂直拆分是指数据表列的拆分，把一张列比较多的表拆分为多张表。**

- **垂直拆分的优点：** 可以使得列数据变小，在查询时减少读取的Block数，减少I/O次数。此外，垂直分区可以简化表的结构，易于维护。
- **垂直拆分的缺点：** 主键会出现冗余，需要管理冗余列，并会引起Join操作，可以通过在应用层进行Join来解决。此外，垂直分区会让事务变得更加复杂；

#### 4. 水平分区

**保持数据表结构不变，通过某种策略存储数据分片。这样每一片数据分散到不同的表或者库中，达到了分布式的目的。 水平拆分可以支撑非常大的数据量。**

​	水平拆分是指数据表行的拆分，表的行数超过200万行时，就会变慢，这时可以把一张的表的数据拆成多张表来存放。举个例子：我们可以将用户信息表拆分成多个用户信息表，这样就可以避免单一表数据量过大对性能造成影响。

​	水平拆分可以支持非常大的数据量。需要注意的一点是：分表仅仅是解决了单一表数据过大的问题，但由于表的数据还是在同一台机器上，其实对于提升MySQL并发能力没有什么意义，所以 **水平拆分最好分库** 。

​	水平拆分能够 **支持非常大的数据量存储，应用端改造也少**，但 **分片事务难以解决** ，跨节点Join性能较差，逻辑复杂。《Java工程师修炼之道》的作者推荐 **尽量不要对数据进行分片，因为拆分会带来逻辑、部署、运维的各种复杂度** ，一般的数据表在优化得当的情况下支撑千万以下的数据量是没有太大问题的。如果实在要分片，尽量选择客户端分片架构，这样可以减少一次和中间件的网络I/O。

**数据库分片的两种常见方案：**

- **客户端代理：** **分片逻辑在应用端，封装在jar包中，通过修改或者封装JDBC层来实现。** 当当网的 **Sharding-JDBC** 、阿里的TDDL是两种比较常用的实现。
- **中间件代理：** **在应用和数据中间加了一个代理层。分片逻辑统一维护在中间件服务中。** 我们现在谈的 **Mycat** 、360的Atlas、网易的DDB等等都是这种架构的实现。

#### 5. 什么是池化设计思想。什么是数据库连接池?为什么需要数据库连接池?

​	池化设计应该不是一个新名词。我们常见的如java线程池、jdbc连接池、redis连接池等就是这类设计的代表实现。这种设计会初始预设资源，解决的问题就是抵消每次获取资源的消耗，如创建线程的开销，获取远程连接的开销等。就好比你去食堂打饭，打饭的大妈会先把饭盛好几份放那里，你来了就直接拿着饭盒加菜即可，不用再临时又盛饭又打菜，效率就高了。除了初始化资源，池化设计还包括如下这些特征：池子的初始值、池子的活跃值、池子的最大值等，这些特征可以直接映射到java线程池和数据库连接池的成员属性中。这篇文章对[池化设计思想](https://mp.weixin.qq.com/s?__biz=Mzg2OTA0Njk0OA==&mid=2247485679&idx=1&sn=57dbca8c9ad49e1f3968ecff04a4f735&chksm=cea24724f9d5ce3212292fac291234a760c99c0960b5430d714269efe33554730b5f71208582&token=1141994790&lang=zh_CN#rd)介绍的还不错，直接复制过来，避免重复造轮子了。

​	数据库连接本质就是一个 socket 的连接。数据库服务端还要维护一些缓存和用户权限信息之类的 所以占用了一些内存。我们可以把数据库连接池是看做是维护的数据库连接的缓存，以便将来需要对数据库的请求时可以重用这些连接。为每个用户打开和维护数据库连接，尤其是对动态数据库驱动的网站应用程序的请求，既昂贵又浪费资源。**在连接池中，创建连接后，将其放置在池中，并再次使用它，因此不必建立新的连接。如果使用了所有连接，则会建立一个新连接并将其添加到池中**。 连接池还减少了用户必须等待建立与数据库的连接的时间。

#### 6.分库分表之后,id 主键如何处理？

因为要是分成多个表之后，每个表都是从 1 开始累加，这样是不对的，我们需要一个全局唯一的 id 来支持。

生成全局 id 有下面这几种方式：

- **UUID**：不适合作为主键，因为太长了，并且无序不可读，查询效率低。比较适合用于生成唯一的名字的标示比如文件的名字。
- **数据库自增 id** : 两台数据库分别设置不同步长，生成不重复ID的策略来实现高可用。这种方式生成的 id 有序，但是需要独立部署数据库实例，成本高，还会有性能瓶颈。
- **利用 redis 生成 id :** 性能比较好，灵活方便，不依赖于数据库。但是，引入了新的组件造成系统更加复杂，可用性降低，编码更加复杂，增加了系统成本。
- **Twitter的snowflake算法** ：Github 地址：https://github.com/twitter-archive/snowflake。
- **美团的[Leaf](https://tech.meituan.com/2017/04/21/mt-leaf.html)分布式ID生成系统** ：Leaf 是美团开源的分布式ID生成器，能保证全局唯一性、趋势递增、单调递增、信息安全，里面也提到了几种分布式方案的对比，但也需要依赖关系数据库、Zookeeper等中间件。感觉还不错。美团技术团队的一篇文章：https://tech.meituan.com/2017/04/21/mt-leaf.html 。