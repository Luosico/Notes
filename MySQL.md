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

