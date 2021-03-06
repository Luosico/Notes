# Oracle

### 一、不同于MySQL的语句

####  拼接列和字符串

- **Oracle**

  **|| 连接运算符**，连接列和字符串，列和列，字符串和字符串

  ```
  select name || age from user;
  select name || ' age is: ' || age from user;
  ```

- **MySQL**

  **contact() 函数**

  ```
  select contact(name,' age is: ',age) from user;
  ```




### 二、创建数据库

​	**Oracle其实没有数据库database这一概念，实际上是表空间，是通过实例给每个用户分配不同的表空间，这样的表空间就是每个用户的数据库。所以直接创建表空间和对应的用户就行了**

​	oracle在创建数据库的时候要对应一个用户，数据库和用户一般一一对应，mysql和sql server 直接通过create databse “数据库名” 就可以直接创建数据库了，而oracle创建一个数据库需要以下三个步骤：

#### 1、创建两个数据库文件

**`test.dbf`和`test_temp.dbf`** **(文件名字自定义)**

```sql
create tableSpace test logging dataFile 'E:\Oracle\oradata\LUOSICO\test.dbf' size 100M autoExtend on next 32M maxSize 500M extent management local;

create temporary tableSpace test_temp tempFile 'E:\Oracle\oradata\LUOSICO\test_temp.dbf' size 100M autoExtend on next 32M maxSize 500M extent management local;
```

#### 2、创建用户

**并与上面创建的文件形成映射关系（这里用户名为`c##lkf`，密码为`123456`）**

```sql
create user c##lkf identified by 123456 default tableSpace test temporary tableSpace test_temp;
# 用户名必须以c##开头
# 有时不需要c##开头
```

​		容器数据库创建新用户并分配表空间时必须在没有PDB的情况下进行或PDB与CDB有相同的表空间的时候进行，否则会报错。如果是在PDB与CDB 有相同表空间的情况下给CDB用户分配表空间，则分配CDB的表空间给用户PDB的表空间并不受影响。而且，CDB用户必须以‘C##’为开头，否则创建不了

```sql
show pdbs;  #查看pdb

#转换容器到pdb
alter session set container=pdb1;
#在pdb中创建与cdb同名的表空间
create tablespace test datafile 'E:\Oracle\oradata\LUOSICO\pdb_test.dbf' size 2m;
create temporary tableSpace test_temp tempFile 'E:\Oracle\oradata\LUOSICO\test_temp.dbf' size 2m;
#回到cdb容器
alter session set container=cdb$root;
```

​		Oracle 12c 引入了CDB(container database)与PDB(plugable database，可插拔数据库 )的概念，**一个CDB可以承载多个PDB**，在这之前，实例与数据库是一对一或多对一关系（RAC）：即一个实例只能与一个数据库相关联，数据库可以被多个实例所加载。而实例与数据库不可能是一对多的关系。当进入ORACLE 12C后，实例与数据库可以是一对多的关系

#### 3、给用户添加权限

```
grant connect,resource,dba to c##lkf ;
grant create session to c##lkf ;
commit;
```

#### 4、删除数据库

```sql
drop tableSpace test including contents and dataFiles;
```

#### 5、删除用户

```SQL
drop user lkf cascade;
```

#### 6、角色

角色是命名的权限的集合，用于分配给用户，可以简化DBA向用户分配权限的工作

**一个用户可以拥有多个角色，一个角色可以分配给多个用户**

```sql
-- 建立角色
create role role_name;

-- 给角色授予权限
grant create table,create view to role_name;

-- 授予角色给用户
grant role_name to user_name;
```

#### 7、授权

```sql
grant object_privileges[(columns...)]
on object
to {user|role option}
[with grant option]; -- 权限可继承
```

#### 8、回收权限

```sql
-- 使用 with grant option授予的用户权限，也一起被撤销
revoke {privileges [,privileges...] | all}
on object
from {user[,user...]|role|public}
[cascade constraints]; --
```



### 三、数据类型

#### number
​	 `number[precision [, scale ]]  precision为精度，scale为小数位数（尺度）`

​		用于存储可能为负值或正值的数值 ，*如果在`NUMBER(p，s)`列中插入数字，并且数字超过精度 `p`，则 Oracle将发出错误。 但是，如果数量超过尺度 `s`，则 Oracle将对该值进行四舍五入*

- number：使用数字的最大范围和精度
- number[p]：定义一个整数
- number[p, s]：定义一个定点数字
- number[p, 0]：定义一个精度为p，小数位数为0的定点数
- number[5, -2]：将数值四舍五入到百



#### float

​	`float ( p )`

​	是number的子类型，其主要目的是促进与ANSI SQL `FLOAT`数据类型的兼容

​	我们只能指定 float 数据类型的精度。不能指定尺度，因为Oracle数据库从数据中解析尺度的。 **float**的最大精度是**126**

​	在`FLOAT`中，精度是二进制位，而在`NUMBER`中精度是十进制数。可以使用以下公式在二进制和十进制精度之间进行转换

```
	P(d) = 0.30103 * P(b)
```



#### char

```
char(length byte)
char(length char)
```

​	存储固定长度的字符串，可存储1到2000字节的字符串，默认使用 **byte**，长度默认值为 **1**，使用 **空格** 填充未使用的字符串



#### nchar

```
nchar(length)
```

​		用于存储固定长度的Unicode字符数据。`NCHAR`的字符集只能是`UTF16`或`UTF8`，在数据库创建时指定为国家字符集，**最大字节长度** 取决于当前的国家字符集。 它是每个字符中最大字符长度和最大字节数的**乘积**



#### varchar2

```
varchar2(max_length byte)
varchar2(max_length char)
```

​		 存储可变字符串，默认使用 **byte**,使用时必须指定最大长度

#### nvarchar2	

```
nvarchar2(max_length)
```

​		存储可变字符串，单位为字符



#### date

​		存储年份(包括世纪)，月份，日期，小时数，分钟数和秒数。 它的范围从公元前4712年1月1日到公元9999年12月31日(共同时代)。 默认情况下，如果未明确使用BCE，则Oracle使用CE日期条目

​		Oracle数据库有其自己的专用格式来存储日期数据。它使用 **7** 个字节的固定长度的字段，每个字段对应于世纪，年，月，日，时，分和秒来存储日期数据。

​		当插入为 YYYY/MM/DD时，自动添加 0时0分0秒，查找时不会显示这个



#### timestamp

```
column_name TIMESTAMP[(fractional_seconds_precision)] 

	fractional_seconds_precision 指定 SECOND 字段小数部分的位数。它的范围从 0 到 9，这意味着可以使用 TIMESTAMP 数据类型来存储到纳秒的精度。如果省略 fractional_seconds_precision，则默认为 6
```

​		用于存储日期和时间数据，包括年，月，日，时，分和秒。另外，它存储小数秒

#### interval

​		用于存储一段时间

#### timestamp with time zone



### 四、自增序列，类似MySQL的自增主键

​	**Oracle里的序列（SEQUENCE），可间接实现自增主键的作用**

#### 1、序列

​	又叫序列生成器，用于提供一系列的数字，开发人员使用序列生成唯一键。每次访问序列，序列按照一定的规律增加或者减少。

​	序列的定义存储在SYSTEM表空间中，序列不像表，它不会占用磁盘空间。

​	序列独立于事务，每次事务的提交和回滚都不会影响序列。

```sql
create sequence sequenceName      //序列名字
increment by 1                    //每次自增1， 也可写非0的任何整数，表示自增，或自减
start with 1                      //以该值开始自增或自减
maxvalue 1.0E20                   //最大值；设置NOMAXVALUE表示无最大值
minvalue 1                        //最小值；设置NOMINVALUE表示无最大值
cycle or nocycle                  //设置到最大值后是否循环；
cache 20                          //指定可以缓存 20 个值在内存里；如果设置不缓存序列，则写NOCACHE
order or noorder                  //设置是否按照请求的顺序产生序列
```



#### 2、步骤

- 创建表

    ```sql
    create table user(
    	id int primary key,
        name varchar(20)
    )
    ```

- 创建自增序列

    ```sql
    create sequence user_sequence
    increment by 1			-- 步长为 1
    start with 1			-- 从 1 开始增加
    nomaxvalue              -- 不设置最大值
    nocycle					-- 一直累加，不循环
    nocache					-- 不设置缓存
    ```

- 创建触发器

    ```sql
    create trigger user_trigger before insert into user
    	for each row
    	begin
    		select user_sequence.nextval into :new.id from dual;
    	end;
    ```

- 插入数据

    ```sql
    insert into table_name(id,name)
    values(user_sequence.nextval,'sally')
    ```

    

### 3、属性

##### nextval

​	返回下一个可用的序列值，即使针对不同的用户，每次也是返回唯一的值，每次会话对应一个值

##### currval

​	返回当前序列值，在其返回序列值之前，必须先分配nextval，即必须在调用nextval后才能使用，它的值对应会话中的nextval的最新值



### 五、索引

```sql
create unique|bitmap index <schema.> index_name on <schema.> table_name
(
	<column_name> | <expression> asc | desc, -- 索引列
    <columm_name> | <expression> asc | decs,
    ...
)
tableSpace <tableSpace_name>
storage <storage_settings>
logging | noLogging
compute statistics
nocompress | compress<nn>
noSort | reverse
partition | global partition<partition_setting>
```

#### 相关说明

1、unique | bitmap：指定unique为唯一值索引，bitmap为位图索引，省略为B-Tree索引

2、<column_name> | <expression>  ASC | DESC：可以对多列进行联合索引，当为expression时即“基于函数的索引”

3、tableSpace：指定存放索引的表空间(索引和原表不在一个表空间时效率更高)

4、storage：可进一步设置表空间的存储参数

5、logging |noLogging：是否对索引产生重做日志(对大表尽量使用noLogging来减少占用空间并提高效率)

6、compute statitstics：创建新索引时收集统计信息

7、nocompress | compress<nn>：是否使用“键压缩”(使用键压缩可以删除一个键列中出现的重复值)

8、noSort |reverse：nosort表示与表中相同的顺序创建索引，reverse表示相反顺序存储索引值

9、partition| nopartition：可以在分区表和未分区表上对创建的索引进行分区

#### 删除索引

```sql
drop index index_name;
```



### 六、约束

#### 1、建表时

```sql
-- 主键
constriant constraint_name primary key(column...)

-- 唯一性
constriant constraint_name unique(column...)

-- 外键
constriant constraint_name foreign key(column) references table_name(column)
		on delete cascade  -- 父表行被删除，字表行被级联删除
		on delete set null -- 父表行被删除，字表行设置为空值null
		
-- 检查
constraint constraint_name check(条件)
```

#### 2、建表后

```sql
-- 禁用约束
alter table table_name disable constraint constraint_name
-- 启用约束
alter table table_name enable constraint constraint_name
-- 删除约束
alter table table_name drop constraint constraint_name

-- 主键
alter table table_name add constraint constraint_name primary key(column...)

-- 唯一性
alter table table_name add constraint constraint_name unique(column...)

-- 外键
alter table table_name add constraint constraint_name foreign key(column) references table_name(column)
		on delete cascade  -- 父表行被删除，字表行被级联删除
		on delete set null -- 父表行被删除，字表行设置为空值null

-- 检查
alter table table_name add constraint constraint_name check(条件)
```



### 七、同义词 Synonym

​	**相当于另一个对象的别名，可以简化对象的访问**

​	**public **使同义词可以由所有用户访问

​	私有同义词的名字不可以与同用户的其他对象同名

```sql
create  [public] synonym for object;
```



### 八、 dual

​	**指的是Oracle中的dual表，是一个单行单列的虚拟表，是与数据字典一起自动创建的一个表，这个表只有1列DUMMY，数据类型为VERCHAR2(1)，dual表中只有一个数据’X’, Oracle有内部逻辑保证dual表中永远只有一条数据**

​	主要用来选择系统变量或求一个表达式的值

```sql
select sysdate from dual
```



### 九、分组函数

#### 1、rollup

```sql
... group by rollup(a,b,c)
```

相当于分组`(a,b,c) (a,b) (a) ()`，并将它们的结果和起来`union all` 

#### 2、cube

```sql
... group by cube(a,b,c)
```

相当于分组`(a,b,c) (a,b) (a,c) (b,c) (a) (b) (c) ()`



### 十、存储过程

#### 1、基本结构

包含三部分：**过程声明**，**执行过程部分**，**存储过程异常**（可写可不写，要增强脚本的容错性和调试的方便性那就写上异常处理）

#### 2、基本实例

```sql
CREATE OR REPLACE PROCEDURE name(
	id in number(10)	
) AS/IS
	变量2 DATE;
	变量3 NUMBER;
BEGIN
	--要处理的业务逻辑
	EXCEPTION    --存储过程异常
END
```

#### 3、is/as

在begin和end之间，存储过程内部使用的各类变量或者常量，需要在 **is/as** 和 **begin** 之间进行定义

#### 4、循环

```sql
FOR a in 10 .. 20 LOOP
  dbms_output.put_line('value of a: ' || a);
END LOOP;
```



```sql
while i<10 loop
	dbms_output.put_line('hello');
end loop;
```



```
LOOP
      x := x + 1;
    
      EXIT WHEN x > 9;
      DBMS_OUTPUT.PUT_LINE('x=' || x);
 END LOOP;
```



#### 5、dbms_output

- enable 

    ​	在serveroutput on的情况下，用来使dbms_output生效(默认即打开)

    ​	**启用serveroutput : set serveroutput on**

    ​	**关闭serveroutput : set serveroutput off**

- disable

    ​	在serveroutput on的情况下，用来使dbms_output失效

- put（）

    ​	将内容写到内存，等到put_line/new_line时一起输出

- put_line(value)

    ​	输出字符（如果缓存中有内容，同时输出）

- new_line

    ​	换行（如果缓存中有内容，同时输出）

- get_line(value, status)

    ​	获取缓冲区的单行信息（返回的内容被存在value中，返回的状态会被存在status）

- get_lines(array, status)

    ​	以数组形式来获取缓冲区的多行信息

#### 6、游标

```sql
-- 获取游标
CURSOR cur_cdd IS SELECT s_id, s_name FROM student;

-- 打开游标游标分配内存，使得它准备取的SQL语句转换成它返回的行
OPEN cur_cdd;

-- 抓取游标中的数据，可用LIMIT关键字来限制条数，如果没有默认每次抓取一条
FETCH cur_cdd INTO id, name;



-- 关闭游标来释放分配的内存
CLOSE cur_cdd;

```

**从游标获取数据**

**<font color='green'>取一次必须调用一次 fetch</font>**

```sql
create or replace procedure myprocedure is
      CURSOR CUR_TEST IS --声明显式游标
             SELECT ECODE,ENAME FROM EMP;
      CUR CUR_TEST%ROWTYPE; --定义游标变量，该变量的类型为基于游标C_EMP的记录

    BEGIN 
      --For 循环
      FOR CUR IN CUR_TEST LOOP
          --循环体
        DBMS_OUTPUT.PUT_LINE('员工编号：'||CUR.ECODE ||'员工姓名：'|| CUR.ENAME);
      END LOOP;

      --Fetch 循环
      OPEN CUR_TEST;--必须要明确的打开和关闭游标
      LOOP 
        FETCH CUR_TEST INTO CUR;
        EXIT WHEN CUR_TEST%NOTFOUND;
        --循环体
        DBMS_OUTPUT.PUT_LINE('员工编号：'||CUR.ECODE ||'员工姓名：'|| CUR.ENAME);
      END LOOP; 
      CLOSE C_EMP;

      --While 循环
      OPEN CUR_TEST;--必须要明确的打开和关闭游标
        FETCH CUR_TEST INTO CUR;
        WHILE CUR_TEST%FOUND LOOP  
          --循环体
          DBMS_OUTPUT.PUT_LINE('员工编号：'||CUR.ECODE ||'员工姓名：'|| CUR.ENAME);

          FETCH CUR_TEST INTO CUR;
        END LOOP;
      CLOSE C_EMP;

END myprocedure;
```



##### 1、游标属性

- %FOUND

    如果DML语句执行后影响有数据被更新或DQL查到了结果，返回true。否则，返回false。

- %NOTFOUND

    如果DML语句执行后影响有数据被更新或DQL查到了结果，返回false。否则，返回true。

- %ISOPEN

    游标打开时返回true，反之，返回false。

- %ROWCOUNT

    返回DML执行后影响的行数。

##### 2、BUIK COLLECT

​	Oracle8i中首次引入了Bulk Collect特性，Bulk Collect会能进行批量检索，会将检索结果结果一次性绑定到一个集合变量中，而不是通过游标cursor一条一条的检索处理。可以在SELECT INTO、FETCH INTO、RETURNING INTO语句中使用BULK COLLECT

- select into 

    查出来一个结果集合赋值给一个集合变量

    ```sql
    select f1,f2 bulk collect into m from table_name;
    ```

- fetch into 

    从一个集合中抓取一部分数据赋值给一个集合变量

    ```sql
    fetch cursor_name bulk collect into m [limit rows]
    --  [LIMIT rows]：可有可无，限制每次抓取的数据量。不写的话，默认每次一条数据
    ```



### 十一、限制查询条数

 **rownum**，都是从1 开始，只要是从1 开始就能返回结果

#### 1、等值

- rownum = 1，返回第一条数据
- rownum = 2 （大于 1 的数），不符合从1开始，不会返回结果

#### 2、不等式

- rownum < 3，返回两条数据，第一条和第二条数据
- rownum > 1，不符合从 1 开始，不会返回结果
- rownum > 0，返回所有数据