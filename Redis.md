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
		incr				将键存储的值加 1，即使是字符串，只要能解释为整数都可以
		decr				将键存储的值减 1
		incrby				将键存储的值加上整数amount
		decrby				将键存储的值减去整数amount
		incrbyfloat			将键存储的值加上浮点数amount
		append				将值value追加到给定键的值的末尾
		getrange			获取偏移量start至end（包含在内）范围内所有字符组成的子串
		setrange			将从start偏移量开始的子串设置为定值，新值有多少改多少
		getbit				将字符串看成二进制位串，并返回位串中偏移量为offset的二进制位的值
		setbit				将字符串的二进制位串的offset位置的二进制位设为value
		bitcount			统计二进制位串中为1的数量，若给定start和end偏移量，则只对偏移范围内统计
		bitop				对一个或多个二进制位串执行位运算，结果存在dest-key里面

- list  
	列表结构可以有序的存储多个字符串
		
		lpush / rpush		将给定值推入列表左/右端		
		lrange				获取列表在给定范围的所有值
		lindex				获取在给定位置上的单个元素
		lpop / rpop			从列表左/右端弹出一个值，并返回弹出的值
		ltrim				只保留从start到end偏移量范围内的元素，包含start和end

		blpop / brpop		从第一个非空列表中弹出最左/右端的元素；若列表全为空，在timeout秒之内阻塞并等待可弹出元素出现
		rpoplpush 			从source-key列表中弹出最右端的元素，并推入dest-key列表最左端，向用户返回这个元素
		brpoplpush			同上，如果source-key为空，那么在timeout秒之内阻塞并等待可弹出元素出现				
		
- set  
	集合通过使用散列表来保证存储的每个字符串都是各不相同的
		
		sadd				将给定元素添加到集合
		srem				如果元素存在，则移除这个元素
		scard				获取集合的成员数
		sismember			检查元素是否存在于集合中
		smembers			返回集合包含的所有元素
		srandmember			随机返回一个或多个元素，若count>0,返回元素不重复，count<0，返回元素可能重复
		spop				随机移除一个元素，并返回这个元素
		smove				如果source-key包含元素item，那么移除item，并将其添加到dest-key中；如果item成功移除，返回1，否则返回0
		sinter				返回同时存在于所有集合的元素
		sinterstore			同上，并将结果存到dest-key
		sunion				并集，返回至少存在于一个集合的元素
		sunionstore			同上，结果存到dest-key
		sdiff				差集，存在第一个集合，但不存在于其他集合
		sdiffstore			t同上，并将结果存到dest-key里面

- zset  
	存储多个键值对，有序集合的键称为成员，成员的值称为分值score,必须是浮点数，是redis唯一一个既可以根据成员访问成员，又可以根据分值以及分值的排列顺序来访问元素的结构
	
		zadd				将一个带有给定分值的成员添加到有序集合中
		zrange				返回排名介于start和stop之间的成员，若给定了可选的withscores选项，那么分值也将返回
		zrem				若存在，移除该成员
		zcard				获取成员数	
		zcount				返回分值位于min和max之间的成员数量
		zincrby				对指点成员的分值加上increment
		zscore				返回成员的分值
		zrank				返回成员在有序集合中的排名
		zrevrank			返回成员的排名，按照分值从大到小排列
		zrevrange			返回给定排名范围内地成员，按照分值从大到小排列
		zrangebyscore		返回分值介于min和max范围内的所有成员
		zrevrangebysocore	同上，安装分值从大到小排列
		zremrangebyrank		移除排名介于start和stop之间的所有成员
		zremrangebyscore	移除分值介于min和max之间的所有成员
		zinterstore			执行类似set的交集运算
		zunionstore			执行类似set的并集运算

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
		hincrby				指定键的整数值加上整数increment
		hincrbyfloat		指定键的浮点数值加上浮点数increment
		hkeys				获取所有的键
		hvals				获取所有键相对应的值
		hlen				获取键的数量
		