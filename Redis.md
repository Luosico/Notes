# <center><font size=7>Redis</font></center>
----
## 1、数据结构<font size=3>
- String (字符串)：字符串、整数、浮点数
- list (列表)：每个节点都是字符串
- set (集合)：不相同的无序的字符串集合
- hash (散列)：包含键值对的无序散列表,字符串和数字值
- zset (有序集合)：字符串成员与浮点数分值之间的有序映射

## 2、常用命令
- 通用命令
	
		del					若键存在，则删除它
		type				返回键所存储值的类型
		rename				修改键的名字
		renamenx			仅当 newkey 不存在时，将 key 改名为 newkey
		move				将当前数据库的键移动到给定的数据库中
		exists				检查键是否存在
		expire				给键设置过期时间，多少秒后过期
		pexpire				设置键的过期时间，以毫秒计
		expireat			设置过期时间，采用Unix时间戳，以秒计
		pexpireat			设置键的过期时间戳，以毫秒计
		persist				移除键的过期时间
		
- String
		
		get					获取存储在给定键中的值
		set   				设置存储在给定键中的值
		del					删除存储在给定键中的值

- list  
	列表结构可以有序的存储多个字符串
		
		lpush / rpush		将给定值推入列表左/右端		
		lrange				获取列表在给定范围的所有值
		lindex				获取在给定位置上的单个元素
		lpop / rpop			从列表左/右端弹出一个值，并返回弹出的值
		
- set  
	集合通过使用散列表来保证存储的每个字符串都是各不相同的
		
		sadd				将给定元素添加到集合
		srem				如果元素存在，则移除这个元素
		scard				获取集合的成员数
		sismember			检查元素是否存在于集合中
		smembers			返回集合包含的所有元素
		sinter				返回给定所有集合的交集
		sunion				并集
		sdiff				差集

- zset  
	存储多个键值对，有序集合的键称为成员，成员的值称为分值score,必须是浮点数，是redis唯一一个既可以根据成员访问成员，又可以根据分值以及分值的排列顺序来访问元素的结构
	
		zadd				将一个带有给定分值的成员添加到有序集合中
		zrange				根据元素在有序排列中所处的位置，来获取多个元素（包括成员和分数）
		zrangebyscore		获取跟定分值范围内的所有元素
		zrem				若存在，移除该成员
		zcard				获取成员数	
		zcount				指定区间的成员数
		zincrby				对指点成员的分值加上increment
		zscore				返回成员的分值

- hash  
	存储多个键值对之间的映射，存储的值为字符串或数字值，可以多数字值执行自增或自减操作

		hset				在散列表里设置键值对，若已存在则更新值，否则新增
		hsetnx				只有当键不存在时才设置键值对
		hmset				设置多个键值对
		hget				获取散列表里给定键的值
		hmget				获取给定所有键的值
		hgetall				获取所有键值对
		hdel				在散列表中若该键存在则移除
		hexists				查看键是否存在
		hincrby				指定键的整数值加上增量increment
		hincrbyfloat		指定键的浮点数值加上增量increment
		hkeys				获取所有的键
		hvals				获取所有键相对应的值
		hlen				获取键的数量
		