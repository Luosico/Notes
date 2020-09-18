# <center><font size=7>Redis</font></center>
----
## 1、数据结构<font size=3>
- String (字符串)：字符串、整数、浮点数
- list (列表)：每个节点都是字符串
- set (集合)：不相同的无序的字符串集合
- hash (散列)：包含键值对的无序散列表,字符串和数字值
- zset (有序集合)：字符串成员与浮点数分值之间的有序映射

## 2、常用命令
### 通用命令

```
	del					若键存在，则删除它
	type				返回键所存储值的类型
	rename				修改键的名字
	renamenx			仅当 newkey 不存在时，将 key 改名为 newkey
	move				将当前数据库的键移动到给定的数据库中
	exists				检查键是否存在
	expire				给键设置过期时间，多少秒后过期并自动删除
	pexpire				设置键的过期时间，以毫秒计
	expireat			设置过期时间，采用Unix时间戳，以秒计
	pexpireat			设置键的过期时间戳，以毫秒计
	persist				移除键的过期时间
	sort				根据给定的选项，进行排序，并返回结果，默认升序
	debug object		查看特定对象的相关信息，查看是否是压缩列表
```

### String

```
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
```

### list  
列表结构可以有序的存储多个字符串
	

```
lpush / rpush		将给定值推入列表左/右端		
lrange				获取列表在给定范围的所有值
lindex				获取在给定位置上的单个元素
lpop / rpop			从列表左/右端弹出一个值，并返回弹出的值
ltrim				只保留从start到end偏移量范围内的元素，包含start和end

blpop / brpop		从第一个非空列表中弹出最左/右端的元素；若列表全为空，在timeout秒之内阻塞并等待可弹出元素出现
rpoplpush 			从source-key列表中弹出最右端的元素，并推入dest-key列表最左端，向用户返回这个元素
brpoplpush			同上，如果source-key为空，那么在timeout秒之内阻塞并等待可弹出元素出现				
```
### set  
集合通过使用散列表来保证存储的每个字符串都是各不相同的
	

```
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
```
### zset  
存储多个键值对，有序集合的键称为成员，成员的值称为分值score,必须是浮点数，是redis唯一一个既可以根据成员访问成员，又可以根据分值以及分值的排列顺序来访问元素的结构

```
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
zinterstore			执行类似set的交集运算，对于相同的成员，新分值为它们分值之和
zunionstore			执行类似set的并集运算，对于相同的成员，新分值为它们分值的最小值
```
### hash  
存储多个键值对之间的映射，存储的值为字符串或数字值，可以多数字值执行自增或自减操作

```
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
```


## 3、发布与订阅（pub/sub）

- 对象
	- 订阅者（listener）: 负责订阅频道（channel）
	- 发送者（publisher）：负责向频道发送二进制字符串消息（binary string message）
	- 频道（channel）
		    subscribe		    订阅一个或多个频道
				unsubscibe		 退订给定的一个或多个频道，如果执行时没有给定任何频道，那么退订所有频道
			publish			    向给定频道发送消息
			psubsribe		   订阅与给定模式相匹配的所有频道
			punsubscribe	 退订给定的模式，若没有给定模式，则退订所有模式
- 缺点
	- 客户端必须一直在线才能接受到消息，断线可能会导致客户端丢失信息
	- 在旧版的Redis里，可能会因为订阅者处理消息的速度不够快而导致发送者输出缓冲区不受控制地增长，变得不稳定甚至崩溃，又或者被管理员杀死；在新版的Redis里，速度缓慢的订阅者可能会被断开连接



## 4、事务

Redis 可以通过 **MULTI，EXEC，DISCARD 和 WATCH** 等命令来实现事务(transaction)功能

- multi命令和exec命令包围的所有命令会一个接一个地执行，直到所有命令都执行完毕。当一个事务执行完毕之后，才会处理其他客户端命令

- Redis事务不是原子性操作，单个Redis命令是原子性操作，事务中某条命令失败不会导致前面已做指令的回滚，也不会造成后续的指令不做

- 事务不能嵌套

		​    multi						标记一个事务块的开始
	​	exec						 执行事务块内的命令
	​	discard					取消该事务，放弃执行事务块内的所有命令，之前执行完成的会恢复到之前，在该命令之后执行redis命令就不是在事务了
	​	watch					  监视一个或多个key，如果在事务执行之前key被其他命令所改动，那么事务将被打断
	​	unwatch				 取消watch命令对key的监视

**Redis 是不支持 roll back 的，因而不满足原子性的（而且不满足持久性）**

Java中Redis事务的实现：

		//开启事务
	    Transaction transaction = jedis.multi();
	    //执行结果
	    Response response = transaction.get("str");
	    //提交事务,List存储事务过程中所有的返回结果
	    List<Object> list = transaction.exec();
	    //必须在事务提交后才能获取Response结果
	    System.out.println(response.get());
	    
	    System.out.println(list.size());
	
	    for(Object o : list){
	        System.out.println(o);
	    }



## 5、键的过期

​	Redis 通过一个叫做**过期字典**（可以看作是hash表）来保存数据过期的时间。过期字典的键指向Redis数据库中的某个key(键)，过期字典的值是一个long long类型的整数，这个整数保存了key所指向的数据库键的过期时间（毫秒精度的UNIX时间戳）。

- 通过Redis的过期时间（expiration）特性来让一个键在给定的时限（timeout）之后自动删除

- 键的过期命令只能为整个键设置过期时间，而没办法为键里面的单个元素设置过期时间
	
		```
		expire				给键设置过期时间，多少秒后过期并自动删除
		pexpire				设置键的过期时间，以毫秒计
		expireat			设置过期时间，采用Unix时间戳，以秒计
		pexpireat			设置键的过期时间戳，以毫秒计
		persist				移除键的过期时间
		ttl					查看给定键距离过期时间还有多少秒
		pttl				查看给定键距离过期时间还有多少毫秒
	```

### 5.1 过期数据的删除策略

#### 	5.1.1 惰性删除

​		只会在取出key的时候才对数据进行过期检查。这样对CPU最友好，但是可能会造成太多过期 key 没有被删除。

#### 	5.1.2  定期删除

​		每隔一段时间抽取一批 key 执行删除过期key操作。并且，Redis 底层会通过限制删除操作执行的时长和频率来减少删除操作对CPU时间的影响

定期删除对内存更加友好，惰性删除对CPU更加友好。两者各有千秋，所以Redis 采用的是 **定期删除+惰性/懒汉式删除**

​	但是，仅仅通过给 key 设置过期时间还是有问题的。因为还是可能存在定期删除和惰性删除漏掉了很多过期 key 的情况。这样就导致大量过期 key 堆积在内存里，然后就Out of memory了。

怎么解决这个问题呢？答案就是： **Redis 内存淘汰机制**

### 5.2  内存淘汰机制

1. **volatile-lru（least recently used）**：从已设置过期时间的数据集（server.db[i].expires）中挑选最近最少使用的数据淘汰

2. **volatile-ttl**：从已设置过期时间的数据集（server.db[i].expires）中挑选将要过期的数据淘汰

3. **volatile-random**：从已设置过期时间的数据集（server.db[i].expires）中任意选择数据淘汰

4. **allkeys-lru（least recently used）**：当内存不足以容纳新写入数据时，在键空间中，移除最近最少使用的 key（这个是最常用的）

5. **allkeys-random**：从数据集（server.db[i].dict）中任意选择数据淘汰

6. **no-eviction**：禁止驱逐数据，也就是说当内存不足以容纳新写入数据时，新写入操作会报错。这个应该没人使用吧！

   **4.0 版本后增加以下两种**：

   7.**volatile-lfu（least frequently used）**：从已设置过期时间的数据集(server.db[i].expires)中挑选最不经常使用的数据淘汰

   8.**allkeys-lfu（least frequently used）**：当内存不足以容纳新写入数据时，在键空间中，移除最不经常使用的 key



## 6、持久化

### 方式 ###
- 快照（snapshotting）:将存在于某一时刻的所有数据都写入硬盘里面
- 只追加文件（AOF）：在执行写命令时，将被执行的写命令复制到硬盘里面
	
#### 快照
- 快照将被写入 dbfilename 选项指定的文件里，并存储在 dir 选项指定的路径上面

- 缺点：Redis、系统和硬件中任意一个崩溃，将会丢失最近一次创建快照之后写入的所有数据

		```
		save				在快照创建完毕之前将不再响应任何其他命令
		bgsave				Redis会创建一个子进程来负责将快照写入硬盘，父进程则继续处理命令请求
		save配置（配置文件中	比如save 60 1000 	从最后一次创建快照之后算起，当“60秒之内有1000此写入”这个条件满足，自动触						发bgsave命令
		shutdown/term		执行save命令，阻塞所有客户端，并在save命令执行完毕后关闭服务器
		sync				Redis服务器之间使用该命令，如果主服务器目前没有在执行bgsave，
							或者主服务器并非刚刚执行完bgsave，那么主服务器就会执行bgsave
	```
#### AOF
- 通过在配置文件的appendonly yes 配置选项来打开
		
		
	```
		always				每个写命令都要同步写入硬盘，这样会严重降低Redis的速度
		everysec			每秒执行一次同步，显式地将多个命令同步到硬盘
		no					让操作系统来决定应该何时进行同步
		bgrewriteaof		通过移除AOF文件中的冗余命令来重写AOF文件，是AOF文件的体积尽可能的小
	```
	
	

## 7、Pipeline（流水线）
- 可以在不使用事务的前提下进行批量操作，减少与服务器的交互次数
	
		```
   	// 开启流水线
	    Pipeline pipeline = jedis.pipelined();
		pipeline.get("key");
   	//只执行同步但不返回结果
       pipeline.sync();
       // 以list的形式返回执行过的命令的结果
       List<Object> result = pipeline.syncAndReturnAll();
   ```



## 8、锁

- **watch**
	
	- 其实就是 **乐观锁**（只会在数据被其他客户端抢先修改了的情况下通知执行了这个命令的客户端，而不会阻止其他客户端对数据的修改）
- **setnx**
	- 只会在键不存在的情况下为键设置值（set not exit）
	
	- 该命令执行后，查询该键没问题，若覆盖该键的值将会失败
		
  		```
   	//获得锁,其实就是通过这个键的存在与否间接的设置锁
     	jedis.setnx("lock","symbol");
   
     	...
     
	      	//释放锁
      	jedis.del("lock");
	   ```
	   
	- 带有超时限制特性，避免锁的持有者崩溃时不会自动释放锁
	
			```
			//获得锁
			jedis.setnx("lock","symblo");
			//设置锁的超时时间
			jedis.expire("lock,10);
		```
- **信号量**



## 9、缓存击穿、缓存穿透、缓存雪崩

### 9.1 缓存雪崩

​	**缓存在同一时间大面积的失效，后面的请求都直接落到了数据库上，造成数据库短时间内承受大量请求**

#### 解决办法

**针对 Redis 服务不可用的情况：**

1. 采用 Redis 集群，避免单机出现问题整个缓存服务都没办法使用。
2. 限流，避免同时处理大量的请求。

**针对热点缓存失效的情况：**

1. 设置不同的失效时间比如随机设置缓存的失效时间。
2. 缓存永不失效。



### 9.2 缓存穿透

​	大量请求的 key 根本不存在于缓存中，导致请求直接到了数据库上，根本没有经过缓存这一层。举个例子：某个黑客故意制造我们缓存中不存在的 key 发起大量请求，导致大量请求落到数据库

#### 解决办法

- **缓存无效 key**

  如果缓存和数据库都查不到某个 key 的数据就写一个到 Redis 中去并设置过期时间，可以解决请求的 key 变化不频繁的情况。如果黑客恶意攻击，每次构建不同的请求 key，会导致 Redis 中缓存大量无效的 key 。很明显，这种方案并不能从根本上解决此问题。如果非要用这种方式来解决穿透问题的话，尽量将无效的 key 的过期时间设置短一点比如 1 分钟

- **布隆过滤器**

  **布隆过滤器说某个元素存在，小概率会误判。布隆过滤器说某个元素不在，那么这个元素一定不在**

  ​	把所有可能存在的请求的值都存放在布隆过滤器中，当用户请求过来，先判断用户发来的请求的值是否存在于布隆过滤器中。不存在的话，直接返回请求参数错误信息给客户端，存在的话才会走下面的流程

  

  **误判的原因**：

  **当一个元素加入布隆过滤器中的时候，会进行哪些操作：**

  1. 使用布隆过滤器中的哈希函数对元素值进行计算，得到哈希值（有几个哈希函数得到几个哈希值）
  2. 根据得到的哈希值，在位数组中把对应下标的值置为 1

  **当我们需要判断一个元素是否存在于布隆过滤器的时候，会进行哪些操作：**

  1. 对给定元素再次进行相同的哈希计算；
  2. 得到值之后判断位数组中的每个元素是否都为 1，如果值都为 1，那么说明这个值在布隆过滤器中，如果存在一个值不为 1，说明该元素不在布隆过滤器中。

  然后，一定会出现这样一种情况：**不同的字符串可能哈希出来的位置相同。** （可以适当增加位数组大小或者调整我们的哈希函数来降低概率）



### 9.3 缓存击穿



## 10、缓存读写模式/更新策略

**下面介绍到的三种模式各有优劣，不存在最佳模式，根据具体的业务场景选择适合自己的缓存读写模式。**

#### 5.1. Cache Aside Pattern（旁路缓存模式）

1. 写：更新 DB，然后直接删除 cache 。
2. 读：从 cache 中读取数据，读取到就直接返回，读取不到的话，就从 DB 中取数据返回，然后再把数据放到 cache 中。

Cache Aside Pattern 中服务端需要同时维系 DB 和 cache，并且是以 DB 的结果为准。另外，Cache Aside Pattern 有首次请求数据一定不在 cache 的问题，对于热点数据可以提前放入缓存中。

**Cache Aside Pattern 是我们平时使用比较多的一个缓存读写模式，比较适合读请求比较多的场景。**

#### 5.2. Read/Write Through Pattern（读写穿透）

Read/Write Through 套路是：服务端把 cache 视为主要数据存储，从中读取数据并将数据写入其中。cache 服务负责将此数据读取和写入 DB，从而减轻了应用程序的职责。

1. 写（Write Through）：先查 cache，cache 中不存在，直接更新 DB。 cache 中存在，则先更新 cache，然后 cache 服务自己更新 DB（**同步更新 cache 和 DB**）。
2. 读(Read Through)： 从 cache 中读取数据，读取到就直接返回 。读取不到的话，先从 DB 加载，写入到 cache 后返回响应。

Read-Through Pattern 实际只是在 Cache-Aside Pattern 之上进行了封装。在 Cache-Aside Pattern 下，发生读请求的时候，如果 cache 中不存在对应的数据，是由客户端自己负责把数据写入 cache，而 Read Through Pattern 则是 cache 服务自己来写入缓存的，这对客户端是透明的。

和 Cache Aside Pattern 一样， Read-Through Pattern 也有首次请求数据一定不再 cache 的问题，对于热点数据可以提前放入缓存中。

#### 5.3. Write Behind Pattern（异步缓存写入）

Write Behind Pattern 和 Read/Write Through Pattern 很相似，两者都是由 cache 服务来负责 cache 和 DB 的读写。

但是，两个又有很大的不同：**Read/Write Through 是同步更新 cache 和 DB，而 Write Behind Caching 则是只更新缓存，不直接更新 DB，而是改为异步批量的方式来更新 DB。**

**Write Behind Pattern 下 DB 的写性能非常高，尤其适合一些数据经常变化的业务场景比如说一篇文章的点赞数量、阅读数量。** 往常一篇文章被点赞 500 次的话，需要重复修改 500 次 DB，但是在 Write Behind Pattern 下可能只需要修改一次 DB 就可以了。

但是，这种模式同样也给 DB 和 Cache 一致性带来了新的考验，很多时候如果数据还没异步更新到 DB 的话，Cache 服务宕机就 gg 了。

