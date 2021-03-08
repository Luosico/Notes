# Mybatis

## 一、SpringBoot部署

  - 在`application.properties`添加mybatis配置、数据源

    ```
    #mybatis
    mybatis.config-location=classpath:mybatis/mybatis-config.xml
    mybatis.mapper-locations=classpath:mybatis/mappers/*.xml
    
    #数据源
    spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
    spring.datasource.url=jdbc:mysql://localhost:3306/seckill?    useUnicode=true&characterEncoding=utf8&serverTimezone=Asia/Shanghai
    spring.datasource.username=root
    spring.datasource.password=*****
    ```
    
- 创建 `mybatis-config.xml（非必须）` 和 `mapper.xml` 文件

- 创建`mapper接口`，并添加`@Repository`注解，使用可直接通过自动注入来完成sql操作

  ```
  	@Autowired
      UserMapper userMapper; 
  
      @RequestMapping(value = "/hello",method = RequestMethod.GET)
      public String hello() throws IOException {
          User user = userMapper.selectUser("admin");
          System.out.println(user.getUserName());
          return "hello";
      }
  ```

- 添加`@MapperScan("com.mapper")`才能扫描到`mapper接口`文件



## 二、`#{}` 和 `${}`

- **#{}** 经过预编译，是安全的；而 **${}** 未经过预编译，可能存在SQL注入风险
- Mybatis官方希望在进行数据库操作的时候将具体的参数作为条件的时候推荐使用#{}，因为这种方式更为安全，防止sql注入，如果使用order by 根据字段排序的这种，推荐使用${}这种方式
    - 使用#{}这种方式的时候会进行预处理并换成转义的字符串，也就是用?代替
    - 使用${}的时候，并不会进行预处理，而是直接将这个值设置进去



## 三、一级缓存和二级缓存

**默认只 开启一级缓存** ，基于 **HashMap** 实现

### 1、一级缓存（SqlSession）

又称 **本地缓存**，存在于 **SqlSession**的生命周期中

#### 1.1 生命周期

​		a、MyBatis在开启一个数据库会话时，会创建一个新的SqlSession对象，SqlSession对象中会有一个新的Executor对象。Executor对象中持有一个新的PerpetualCache对象；当会话结束时，SqlSession对象及其内部的Executor对象还有PerpetualCache对象也一并释放掉。

　　b、如果SqlSession调用了close()方法，会释放掉一级缓存PerpetualCache对象，一级缓存将不可用。

　　c、如果SqlSession调用了clearCache()，会清空PerpetualCache对象中的数据，但是该对象仍可使用。

　　d、SqlSession中执行了任何一个update操作(update()、delete()、insert()) ，都会清空PerpetualCache对象的数据，但是该对象可以继续使用

#### 1.2 查询是如何利用缓存的

​	在同一个 SqlSession 中查询时，MyBatis会把执行的 **方法 **和 **参数** 通过算法生成缓存的键值，将键值和查询结果存入一个 **Map** 对象中，如果同一个SqlSession中执行的方法和参数完全一致，那么通过算法会生成相同的键值，当Map缓存对象中已经存在该键值时，则会返回缓存中的对象。

#### 1.3 问题

​	对于重复的两次完全相同的查询，当第一次查询结束后，修改了查询结果，对于第二次查询，依然会直接使用修改后的查询结果（不是预期结果），而不是重新到数据库中查询

​	**可通过设置 flushChache="true"来避免这种情况的发生**

#### 1.4 不使用一级缓存

设置 **flushChache="true"**

这会在查询数据前清空当前的一级缓存，因此每次都会重新到数据库中查询数据



### 2、二级缓存

存在于 **SqlSessionFactory** 的生命周期中

SqlSessionFactory层面上的二级缓存默认是不开启的，二级缓存的开启需要进行配置，实现二级缓存的时候，MyBatis要求返回的POJO必须是可序列化的。 也就是要求实现Serializable接口，配置方法很简单，只需要在映射XML文件配置就可以开启缓存了 `<cache/> `，如果我们配置了二级缓存就意味着：

- 映射语句文件中的所有select语句将会被缓存。
- 映射语句文件中的所欲insert、update和delete语句会刷新缓存。
- 缓存会使用默认的Least Recently Used（LRU，最近最少使用的）算法来收回。
- 根据时间表，比如No Flush Interval,（CNFI没有刷新间隔），缓存不会以任何时间顺序来刷新。
- 缓存会存储列表集合或对象(无论查询方法返回什么)的1024个引用
- 缓存会被视为是read/write(可读/可写)的缓存，意味着对象检索不是共享的，而且可以安全的被调用者修改，不干扰其他调用者或线程所做的潜在修改。



## 四、Mapper.xml

### 1、组成

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<!--对应接口的全限定名称-->
<mapper namespace="com.mapper.UserMapper">
    <insert id="insertUser">
        insert into user(userName,userPassword) values(#{userName},#{userPassword})
    </insert>

    <select id="selectUserName" resultType="Integer">
        select count(userName) from user where userName=#{userName}
    </select>
    <select id="selectUser" resultType="java.lang.Integer">
        select count(userName) from user where userName=#{userName} and userPassword=#{userPassword}
    </select>
</mapper>
```



- **mapper**：XML的根元素

  - **namespace**：定义当前XML的命名空间，一般写对应接口的全限定名称

- **select/insert/update/delete**：`slq`语句放在这个标签中

  - **id**：定义当前`sql`的唯一的 `id`

- **resultType**：定义返回类型，没用`typeAliases`配置包的别名，这里就需要写上类的全限定名称

- **resultMap**：设置返回值的类型和映射关系

  - **id**：唯一，对应sql语句的id
  - **type**：配置查询列所映射到的Java对象类型
  - **extends**：可配置当前resultMap继承其他的resultMap，属性值为继承resultMap的id

  ```xml
  <mapper namespace="com.mapper.UserMapper">
      
      <resultMap id="userMap" type="com.luosico.mapper.User">
      	<id property="id" column="id"/>
          <result property="username" column="user_name"/>
          <result property="img" column="img" jdbcType="BLOB"/>
      </resultMap>
      
      <select id="selectUser" resultMap="userMap">
          select * from user where userName=#{userName}
      </select>
  </mapper>
  ```

  

### 2、自增主键

#### 2.1 支持主键自增的数据库，如 MySQL

  MyBatis会使用JDBC的 getGeneratedKeys 方法来取出由数据库内部生成的主键，获得主键值后将其赋值给 keyproperty 配置的 id 属性。当设置多个属性时，使用逗号隔开，这种情况下通常还需要设置 keyColumn 属性

  ```xml
  <insert id="insertUser" useGeneratedKeys="true" keyProperty="id">
  	insert into user(
      	user_name,
      	password,
      	create_time)
      values(
      	#{userName},
      	#{password},
      	#{createTime, jdbcType=TIMESTAMP }
      )
  </insert>
  
  # 另一种写法
  <insert id="insertUser">
  	insert into user(
      	user_name,
      	password,
      	create_time)
      values(
      	#{userName},
      	#{password},
      	#{createTime, jdbcType=TIMESTAMP }
      )
  </insert>
  
  <selectKey keyColumn="id" resultType="long" keyProperty="id" order="AFTER">
  	select last_insert_id()
  </selectKey>
  ```

  #### 2.2 不支持主键自增的数据库，如 Oracle

  先使用序列得到一个值，然后将这个值赋给id，再将数据插入数据库

  ```xml
  <insert id="insertUser">
  	insert into user(
      	id,
      	user_name,
      	password,
      	create_time)
      values(
      	#{id},
      	#{userName},
      	#{password},
      	#{createTime, jdbcType=TIMESTAMP }
      )
  </insert>
  
  <selectKey keyColumn="id" resultType="long" keyProperty="id" order="BEFORE">
  	select SEQ_ID.nextval from dual
  </selectKey>
  ```

  **order** 属性的设置和使用的数据库有关。MySQL是 `AFTER`, Oracle是 `BEFORE`



## 五、动态SQL

### 5.1 if

```xml
 select 
 	id,
 	name
 from 
 	user
 where 
 	1=1
 	<if test="name!=null and name like '%hello%'">
 		and name = #{name}
 	</if>
 	<if test="id!=null and id like '123'">
 		and id = #{id}
 	</if>
```



### 5.2 choose

一个choose中至少有一个when，0或1个otherwise

```xml
	select 
		id,
		name
	from 
		user
	where 
		1=1
	<choose>
		<when test=" id != null ">
			and id = #{id}
		</when>
		<when test=" name != null ">
			and name = #{name}
		</when>
		<otherwise>
		and 1=2
		</otherwise>
	</choose>
```



### 5.3 trim

where和set 都属于 trim 的一种具体用法，底层是通过 `TrimSqlNode` 实现的

#### 5.3.1  where

如果该标签包含的元素中有返回值，就插入一个 `where`；如果where后面的字符串是以 `and` 和 `or` 开头的，就将它们剔除，当 if 条件都不满足时，where 元素中没有内容，所以在 SQL 中不会出现 where

```xml
	<select>
		select 
			id,
			name
		from 
			user
		<where>
			<if test="name!=null">
            	and name = #{name}
        	</if>
        	<if test="id!=null">
        		and id = #{id}
        	</if>
		</where>
	</select>
```

#### 5.3.2  set

 如果该标签包含的元素中有返回值，就插入一个 `set`；如果 set 后面的字符串是以**逗号**结尾的，就将这个逗号剔除

```xml
<update>
	update
  		user
    <set>
    	<if test="name!=null and name!=''">
        	name = #{name},
        </if>
        <if test = "id!=null and id!=''">
        	id = #{id},
        </if>
        id = #{id},
    </set>
    where id = #{id}
</update>
```

#### 5.3.3 trim标签属性

- prefix：当trim元素内包含内容时，会给内容增加prefix指定的前缀

- prefixOverrides：当trim元素内包含内容时，会把内容中匹配的前缀字符串去掉

- suffix：当trim元素内包含内容时，会给内容增加suffix指定的后缀

- suffixOverrides：当trim元素内包含内容时，会把内容中匹配的后缀字符串去掉

    ![IMG_0700](https://raw.githubusercontent.com/Luosico/Typora/main/img/20210308144821.JPG)



### 5.4 foreach

**`foreach` 可以对数组、Map 或实现了 Iterable 接口（如List、Set）的对象进行遍历**

数组在处理时会转换为List对象，Map就还是Map对象

因此foreach遍历的对象可以分为：Iterable 类型和 Map 类型

```xml
<select>
	select
    	id,
    	name
    from
    	user
    where id in
    <foreach collection="list" open="(" close=")" separator="," item="id" index="i">
    	#{id}
    </foreach>
</select>
```



#### 5.4.1 属性

![IMG_0701](https://raw.githubusercontent.com/Luosico/Typora/main/img/20210308145410.JPG)





### 5.5 bind

可以使用 OGNL 表达式创建一个变量并将其绑定到上下文中

可以用来试用不容数据库的SQL语句