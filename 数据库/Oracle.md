# Oracle

### 不同于MySQL的语句

- ####  拼接列和字符串

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




### 创建数据库





### 数据类型

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



#### timestamp

```
column_name TIMESTAMP[(fractional_seconds_precision)] 

	fractional_seconds_precision 指定 SECOND 字段小数部分的位数。它的范围从 0 到 9，这意味着可以使用 TIMESTAMP 数据类型来存储到纳秒的精度。如果省略 fractional_seconds_precision，则默认为 6
```

​		用于存储日期和时间数据，包括年，月，日，时，分和秒。另外，它存储小数秒

#### interval

​		用于存储一段时间

#### timestamp with time zone

