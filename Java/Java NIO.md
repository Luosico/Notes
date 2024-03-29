# 									<font size=7>Java NIO</font>

## 1、 通道 Channel
- **作用**：通道将缓冲区的数据块移入或移出到各种I/O源，如文件、socket、数据报等。
- **与流的区别**：
	
	- 流是基于字节的，按顺序一个字节接一个字节地传动数据，处于性能考虑，也可以传送字节数组，仍符合
	- 通道是基于块的，传送的是缓冲区中的数据块，这些字节必须已经存储在缓冲区中，而且一次读/写一个缓冲区的数据
- **通道类**: SocketChannel(TCP)、ServerSocketChannel(TCP)、DatagramChannel(UDP)、FileChannel(不能非阻塞，不能使用selector) 等
- **SocketChannel**
	
	- **作用**：可以读写TCP Socket。数据必须编码到 **ByteBuffer** 对象中来完成读/写。每个SockerChannel都与一个对等端Socket相关联。		
	
		```java
		//通过静态open方法创建SocketChannel对象
		ServerChannel channel = SocketChannel.open();
		ServerChannel channel = SocketChannel.open(SocketAddress socketAddress)
		
		//设置是否阻塞（默认阻塞）
		channel.configureBlocking(false);//非阻塞
		
		//阻塞模式下，阻塞直到连接建立成功或出现异常
		//非阻塞模式下，会立即返回，甚至在连接建立之前就会返回，若立即建立成功，返回true，否则返回false
		channel.connect();
		
		//非阻塞模式下，必须调用这个方法，若连接建立成功返回ture,否则false
		//阻塞模式立即返回true
		channel.finishConnect();
		
		//连接打开时返回true
		channel.isConnected()
		//连接仍在建立但尚未打开时返回true
		channel.isConnectionPending();
		
		```
		
	- **读写数据**：必须使用 **ByteBuffer** 缓冲区  
		
		```java
		//通道会用尽可能多的数据填充缓冲区，然后返回放入的字节数
		//流结尾返回-1
		//没数据时，阻塞会等待，非阻塞返回0
	channel.read(ByteBuffer buffer);
		
		//散布：从一个源填充多个缓冲区
		//填满当前buffer后才能向另一个buffer写入数据
	channel.read(ByteBuffer[] buffers);
		//从位于offset的缓冲区开始，填充length个缓冲区
		channel.read(ByteBuffer[] buffers,int offset,int length);
		
		//非阻塞时，不能保证会写入缓冲区的全部内容，不过根据缓冲区基于游标的特性可以保证将其排空
		channel.write(ByteBuffer buffer);
		
		//聚集：将多个缓冲区的数据写入到一个通道
		//填满当前buffer后才能向另一个buffer写入数据
		channel.write(ByteBuffer[] buffers);
		channel.write(ByteBuffer[] buffers,int offset,int length);
		
		//关闭
		channel.close();
		//判断通道是否关闭
		channel.isOpen();
		
		```
- **ServerSocketChannel**
	
	- 作用：只能接受入站连接，无法读取、写入或连接ServerSocketChannel
	
	```java
	   	//获取ServerSocketChannel对象
	   	ServerSockerChannel serverChannel = ServerSocketChannel.open();
	   	//获得相应对等端(peer)的ServerSocket
	   	ServerSocket serverSocket = serverChannle.socket();
	   	//绑定端口，两种方式
	 	serverChannel.bind(InetSocketAddress address);
	 	serverSocket.bind(InetSocketAddress address);
	```
	- 接受连接
	```java
		//阻塞模式下，一直等待直到接受的连接建立成功
		//非阻塞模式下,没有入站连接立即返回 null
		SocketChannel socketChannel = serverChannel.accept();
	```



## 2、异步通道 AsynchronousChannel

- **异步通道类**：AsynchronousSocketChannel、AsynchronousServerSocketChannel等
- **差别**：异步通道会立即返回，甚至在I/O完成之前就会返回，所读写的数据会由Future或CompletionHandle进一步处理，connect()和accept()方法也会异步执行，并返回Future。这里不使用选择器

	```java
		AsynchronousSocketChannel socketChannel = AsynchronousSocketChannel.open(InetSocketAddress address);
		Future<Void> connected = socketChannel.connect();
	ByteBuffer buffer = ByteBuffer.allocate(50);
		//等待连接完成
		connected.get();
		
		//从连接读取
		Future<Integer> future = socketChannel.read(buffer);
		
		//做其他工作
		...
		    
		//等待读取完成
		future.get();
		
		//回绕并排空缓冲区
		buffer.flip();
		//将通道转换成标准输出流
		WritableByteChannel out = Channels.newChannel(System.out);
		out.write(buffer);
	```



## 3、Channels类

- 作用：简单的工具类，可以将传统的基于I/O的流、阅读器和书写器包装在通道中，也可以从通道中转换出来

	```java
		//获取Inpustream
		InputStream in = Channels.newInputStream(channel);
	```



## 4、选择器 Selector

- **多路复用**

- **允许单个线程处理多个Channel，Channel必须处于非阻塞模式下**

- **作用**：能够选择读写时**不阻塞**的Socket,通过将不同的通道注册到一个Selector对象，每个通道都将分配有一个SelectionKey
	
	**注意是非阻塞通道**
	
	```java
	//获取Selector对象
	Selector selector = Selector.open();
	
	//在通道绑定Selector
	//用 | 绑定多个事件
	SelectionKey selectionKey = channel.register(selector,SelectionKey.OP_ACCEPT | 	SelectionKey.OP_CONNECT);
	//第三个参数是附件，通常用于存储连接的状态
	SelectionKey selectionKey = channel.register(selector,SelectionKey.OP_WRITE,Object attach);
	
	//非阻塞选择，如果当前没有准备好要处理的连接，立即返回
	selector.selectNow(); //return int;
	
	//阻塞选择，一直等待直到至少有一个注册的通道准备好可以进行处理
	selector.select(); //return int 自上次调用select()方法后有多少通道变成就绪状态;
	//返回0前只等待不超过timeout毫秒
	selector.select(long timeout);
	
	//当直到有通道已经准备好，获取就绪通道
	Set<SelectionKey> set = selector.selectedKeys();
	
	//某个线程调用select()方法后阻塞了，该方法会打断中断
	selector.wakeUp();
	
	//关闭选择器
	selector.close();
	```
	
- **SelectionKey**
	
	- SelectionKey对象相当于通道的指针，还可以保存一个对象附件，一般会存储通道的连接状态
		
	- 一个SelectionKey对应一个Channel
		
		```java
		//测试SelectionKey对象能进行哪些操作
		public final boolean isAcceptable()
		public final boolean isConnectable()
		public final boolean isReadable()
		public final boolean isWritable()
		```
		
		
		
		```java
		//获取通道
		SelectableChannel channel = selectionKey.channel();
		//设置附件
		selectionKey.attach(buffer);
		//获取附件 attachment
		Object attacher = selectionKey.attachment();
		
		//撤销通道的注册
		selectionKey.cancle();
		
		```

- 实例

  ```java
  Selector selector = Selector.open();
  //设置非阻塞
  channel.configureBlocking(false);
  //绑定channel
  SelectionKey key = channel.register(selector, SelectionKey.OP_READ);
  while(true) {
    int readyChannels = selector.select();
    if(readyChannels == 0) continue;
    Set selectedKeys = selector.selectedKeys();
    Iterator keyIterator = selectedKeys.iterator();
    while(keyIterator.hasNext()) {
      SelectionKey key = keyIterator.next();
      if(key.isAcceptable()) {
          // a connection was accepted by a ServerSocketChannel.
      } else if (key.isConnectable()) {
          // a connection was established with a remote server.
      } else if (key.isReadable()) {
          // a channel is ready for reading
      } else if (key.isWritable()) {
          // a channel is ready for writing
      }
      //Selector不会自己从已选择键集中移除SelectionKey实例。必须在处理完通道时自己移除。下次该通道变成就绪时，Selector会再次将其放入已选择键集中
      keyIterator.remove();
    }
  
  ```

  

## 5、缓冲区 Buffer

- 除boolean外，java所有基本数据类型都有特定的Buffer子类，如：ByteBuffer、CharBuffer等

- 每个缓冲区记录信息的4个关键部分：
	- 位置（position）
		- 缓冲区中将读取或写入的下一个位置，从0开始计，最大值等于容量
	- 容量（capacity）
		- 可以保存的元素的最大数目，在创建缓冲区时设置，此后不能改变
	- 限度（limit）
		- 可访问数据的末尾位置
	- 标记（mark）
		- 缓冲区中客户端指定的索引
		- mark()：将标记设置为当前位置
		- reset（）：将当前位置设置为所标记的位置  
	
- 公共方法
	- clear():将位置设为0，并将限度设置为容量，实际并没有删除数据，还能读取
	- rewind()：将位置设为0，但不改变限度
	- flip()：将限度设为当前位置，位置设为0
	- remaining()：返回当前位置与限度之间的元素数
	- hasRemaining()：若剩余元素大于0，返回true，否则返回false
	
- 创建缓冲区
	- 空的缓冲区由分配（allocate）方法创建，预填充数据的缓冲区由包装（wrap）方法创建
	- allocate()
		- 常用语输入
		
		- 返回一个指定容量的空缓冲区，位置为0
		
		- 它创建的缓冲区基于Java数组实现，可以通过array()和arrayOffset()来访问，修改会反映到缓冲区中，反之亦然，实际上暴露了缓冲区的私有数据，谨慎使用
		
			```java
			ByteBuffer buffer = ByteBuffer.allocate(100);
			byte[] array = buffer.array();
			```
		
	- allocatDirect()
		
		- 直接分配，不为缓冲区创建后备数组
	- wrap()
		- 常用与输出		
		
			```java
			byte[] date = "Hello".getBytes();
			ByteBuffer buffer = ByteBuffer.wrap(data);
			```
	
- 填空和排空
	
	```java
	buffer.put(byte)  //向缓冲区写入一个数据，并位置加1
	buffer.get(int)  //从缓冲区读取一个数据，并位置减1
	
	buffer.get(byte[] dst);
	buffet.get(byte[] dst,int offset,int length);	//从缓冲区中向数组的offset位置写入length长度的数据
	
	buffer.put(byte[] array);
	buffer.put(byte[] array,int offset,int length);	//从数组的offset位置读取length长度到缓冲区
	
	```
	
- 改变字节顺序(默认为大端模式，big-endian)
	
	```java
	buffer.order(ByteOrder.LITTLE_ENDIAN)； //小端模式
	buffer.order(IntOrder.LITTLE_ENDIAN)
	```
	
- 视图缓冲区
	- 可以将当前缓冲区转换成其他类型的缓冲区
		
		```java
		IntBuffer intbuffer = byteBuffer.asIntBuffer();
		FloatBuffer floatBuffer = byteBuffer.asFloatBuffer();
		
		```
		
	- 非阻塞模式不能保证缓冲区在排空后仍能以int、double、或char等类型的边界对齐。向非阻塞通道写入一个ine或double的半个字节是完全可能的
	
- 压缩缓冲区
	- 压缩时将缓冲区中所有剩余的数据移到缓冲区的开头，位置设置为数据末尾
	
			buffer.compact()
	
- 复制缓冲区
	- 返回值不是克隆，复制的缓冲区共享相同的数据，修改一个缓冲区中的数据会反映到其他缓冲区
	- 尽管共享相同的数据，但每个缓冲区都有独立的标记、限度和位置
		
			ByteBuffer duplicateBuffer = buffer.duplicate();
	
- 分片缓冲区
	- 是原缓冲区的一个子序列，分片的起始位置是原缓冲区的当前位置
		
			ByteBuffer sliceBuffer = buffer.slice();
	
- 缓冲区相等的条件，即`buffer1.equals(buffer2)==true`
	- 是相同类型的缓冲区
	- 缓冲区中剩余元素个数相同，即position-limit
	- 相同相对位置上的剩余元素彼此相等