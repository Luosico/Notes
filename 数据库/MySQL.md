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

