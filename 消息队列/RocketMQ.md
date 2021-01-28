# RocketMQ

## 一、消息队列

### 定义

**从广义上讲是一种消息队列服务中间件，提供一套完整的信息生产、传递、消费的软件系统，采用RPC进行通信**

### 作用

- 削峰填谷
- 程序间解耦
- 异步处理
- 数据的最终一致性



## 二、组件

- Producer
- Consumer
- Broker
- Namesrv



## 三、Producer

### 定义

#### 	生产者

​	发送消息的一方

#### 	生产者组

​	一个逻辑概念，包含多个生产者实例，在使用生产者实例的时候需要指定一个组名。一个生产者组可以产生多个**Topic**的消息

#### 生产者实例

​	一个生产者组部署了多个进程，每个进程对应一个生产者实例

#### Topic

​	主题名字，一个Topic有 **若干个Queue** 组成



### 生产者类型

#### DefaultMQProducer

生产 **普通消息、顺序消息、单向消息、批量消息、延迟消息**

#### TransactionMQProducer

生产 **事务消息**



### 消息类型

| 消息类型     | 功能                                                         |
| ------------ | ------------------------------------------------------------ |
| 普通消息     | 也称并发消息，消息没有顺序，生产消息都是并行进行，效率最高   |
| 分区有序消息 | 把Topic消息分为多个分区保存和消费，一个Topic分区只有一个Queue，一个分区内的消息就是传统的队列，遵循FIFO（先进先出）原则 |
| 全局有序消息 | 把Topic的分区数设为 1，相当于该Topic中的消息是单分区，所有消息都遵循FIFO原则 |
| 延迟消息     | 消息发送后，消费者要在一定时间后，或者指定某个时间点才可以消费，消息暂存在Broker中，由Broker的定时投递任务投递 |
| 事务消息     | 主要涉及分布式事务，即需要保证在多个操作同时成功或同时失败时，消费者才能消费消息 |
|              |                                                              |



### 生产者高可用

#### 客户端保证

##### 重试机制

​	设置消息发送到Broker的重试次数，默认为2次，加上正常发送的1次，总共有3次发送机会

- retryTimesWhenSendFailed
- retryTimesWhenSendAsyncFailed

##### 客户端容错

​	RocketMQ Client维护了一个“Broker-发送延迟”关系，根据这个关系选择一个发送延迟级别较低的 Broker 来发送消息，剔除已经宕机、不可用或者发送延迟级别较高的Broker，尽量保证消息的正常发送

**sendLatencyFaultEnable**

​	发送延迟容错开关，**默认为关闭**，如果打开会触发发送延迟容错机制来选择发送Queue（即Broker）



#### Broker端保证

##### 同步复制

​	消息发送到 **Master Broker** 后，同步到 **Slave Broker** 才算发送成功

##### 异步复制

​	消息发送到 **Master Broker**，即为发送成功



### 实例

#### 普通消息

```java
DefaultMQProducer producer = new DefaultMQProducer("ProducerGroupName");
producer.setNamesrvAddr("namesrv addresses");
producer.setRetryTimesWhenSendAsyncFailed(2);
Producer.start();

Message msg = new Message("TopicName","TagName","keysName","Hello World".getBytes(RemotingHelper.DEFAULT_CHARSET));

SendResult sendResult = producer.send(msg);
System.out.printf("%s%n",sendResult);

producer.shutdown();
```



#### 顺序消息

​	根据 **HashKey** 将消息发送到指定分区中，即指定的Queue中，每个分区中的消息都是有序的，即分区有序。若 Topic 的分区数为 1 ，即全局有序。

​	**发送消息必须是单线程，多线程不再有序**

```java
DefaultMQProducer producer = new DefaultMQProducer("ProducerGroupName");
producer.setNamesrvAddr("namesrv addresses");
producer.setRetryTimesWhenSendAsyncFailed(2);
Producer.start();

Message msg = new Message("TopicName","TagName","keysName","Hello World".getBytes(RemotingHelper.DEFAULT_CHARSET));

Integer hashKey = 123;

SendResult sendResult = producer.send(msg,new MessageQueueSelector(){
    @override
    public MessageQueue select(List<MessageQueue> mqs, Message msg,Object arg){
        Integer id = (Integer) arg;
        int index = id % mqs.size();
        return mqs.get(index);
    }
}, hashKey);

System.out.printf("%s%n",sendResult);

producer.shutdown();
```



#### 延迟消息

目前开源版本支持 **18** 个延迟级别，**“1s 5s 10s 30s 1m 2m 3m 4m 5m 6m 7m 8m 9m 10m 20m 30m 1h 2h”**，不支持自定义设置延迟时间

Broker 在接收消息后，将消失保存在 **SCHEDULE_TOPIC_XXXX**(XXXX为TopicName)的 Topic 中，由 Broker 端的定时投递任务投递给消费者

```java
...

//创建消息
Message msg = new Message("TopicName","TagName","Hello World".getBytes(RemotingHelper.DEFAULT_CHARSET));
    
//设置延迟级别
msg.setDelayLevel(4);
...
```



#### 事务消息

先发送到对消费者不可见的 Topic 中。当事务消息被生产者提交后，会被二次投递到原始 Topic 中

步骤：

1.  用户发送一个 **Half 消息 **到 Broker，Broker设置 queueOffset=0，即对消费者不可见
2.  用户本地事务处理成功，发送一个 Commit 消息到 Broker，Broker 修改 queueOffset 为正常值，达到重新投递的目的，此时消费者可以正常消费；如果本地事务处理失败，那么将发送一个 **Rollback 消息** 给 Broker，Broker将删除 Half 消息 

**Broker会定期回查生产者，确认生产者本地事务的执行状态，在决定是提交、回滚还是删除 Half 消息**

```java
//初始化事务消息生产者
TransactionMQProducer producer = new TransactionMQProducer("ProducerGroupName");
//配置生产者的各个参数和Broker回调，检查本地事务处理并启动生产者
producer.setCheckThreadPoolMinSize(2);
producer.setCHeckThreadPoolMaxSize(2);
producer.setCheckRequestHoldMax(2000);

producer.setTransactionCheckListener(new TransactionCheckListener(){
    private AtomicInteger transactionIndex = new AtomicInteger(0);
    @Override
    public LocalTransactionState checkLocalTransactionState(Message msg){
		System.out.printf("server checking TrMsg %s%n", msg);
        
        int value = transationIndex.getAndIncrement();
        if((value % 6) == 0){
			throw new RuntimeException("Could not find db");
        }else if((value % 5) == 0){
            return LocalTransactionState.ROLLBACK_MESSAGE;
        }else if((value % 4) == 0){
			return LocalTransactionState.COMMIT_MESSAGE;
        }
        
        return LocalTransactionState.UNKNOW;
    }
});

producer.start();

//设置本地事务处理器，发送消息
Message msg = new Message("TopicName","TagName","KeyName","Hello World".getBytes(RemotingHelper.DEFAULT_CHARSET));
SendResult sendResult = producer.sendMessageInTransaction(msg, new LocalTransactionExecuter(){
    private AtomicInteger transactionIndex = new AtomicInteger(1);
    
    @Override
    public LocalTransactionState executeLocalTransactionBranch(final Message msg, final Object arg){
		int value = transactionIndex.getAndIncrement();
        
        if(value == 0){
            throw new RuntimeException("Could not find db");
        }else if((value % 5) == 0){
            return LocalTransactionState.ROLLBACK_MESSAGE;
        }else if((value % 4) == 0){
            return LocalTransactionState.COMMIT_MESSAGE;
        }
        
        return LocalTransactionState.UNKNOW;
    }
},null);
```



#### 单向消息

生产者只管发送过程，不管发送结果，主要用于日志传输等消息允许丢失的场景

```java
...

Message msg = new Message(...);

producer.sendOneWay(msg);
```



#### 批量消息

- 消息最好小于 1MB
- 同一批批量消息的Topic、waitStoreMsgOK 属性必须一致
- 批量消息不支持延迟消息

```java
...

List<Message> messages = new ArrayList<>();
messages.add(msg1);
messages.add(msg2);

producer.send(messages);
prducer.shutdown();
```





## 四、Consumer

### 定义

#### 消费者

消费者一般指获取消息、转发消息给业务代码处理的一系列代码实现

#### 消费者组

一个逻辑概念，在使用消费者时需要指定一个组名。一个消费者组可以订阅多个Topic

#### 消费者实例

一个消费者组程序部署了多个进程，每个进程都可以称为一个消费者实例

#### 订阅关系

一个消费者组订阅一个 Topic 的某一个 Tag。一个消费者组中的实例订阅的 Topic 和 Tag 必须完全一致，否则就是订阅关系不一致，会导致消费消息紊乱



### 消费模式

#### 集群消费模式

RocketMQ的默认消费模式，**消息进度**都保存在 Broker 端

适合对消息没有顺序要求的场景，比如异步通信、削峰等

#### 广播消费模式模式

全部的消息都是广播发送，即消费者组中的全部消费者实例将消费整个 Topic 的全部消息，**消费进度**保存在客户端机器的文件中

适合每个消费者实例都需要通知的场景，比如刷新应用服务器中缓存



### 消息的可靠性

#### 重试-死信机制

消费过程可分为3个阶段：**正常消费、重试消费和死信**，对应**正常Topic 、重试Topic、死信Topic**

##### 正常Topic

正常消费者订阅的Topic名字

##### 重试Topic

出现各种意外导致的消息消费失败，那么消息会被自动保存到重试Topic中，格式为”%RETRY%消费者组“，有16次重试的机会，在订阅的时候会自动订阅这个Topic

##### 死信Topic

格式为”%DLQ%消费者组名“，当重试**16次**失败后，消息会被自动保存到这个Topic中，进入这个Topic中消息不能再次消费

#### Rebalance机制

用于在发生Broker掉线、Topic扩容和缩容、消费者扩容和缩容等变化时，自动感知并调整自身消费，以尽量减少甚至避免消息没有被消费



### 消费者类型

#### DefaultMQPullConsumer

用户主动从 Broker 中 Pull 消息和消费消息，提交消费位点

#### DefaultMQPushConsumer

通过Pull服务将消息拉取到本地，再通过 Callback 的形式，将本地消息 Push 给用户的消费代码



### 消息进度保存

也叫 **消息进度持久化**

分为：远程位点管理 和 本地位点管理

- LocalFileOffsetStore
- RemoteBrokerOffsetStore

支持：

- 定时持久化：通过定时任务来定时持久化
- 不定时持久化：也叫 Pull-And-Commit，在执行Pull方法的同时，把队列最新消费位点信息发送给Broker



### 消费方式

#### Pull方式

Client端循环的从Server端拉取消息，主动权在Client手里

用户主动Pull消息，自主管理位点，可以灵活地掌握消费进度和消费速度，适合流计算、消费特别耗时等特殊的消费场景。缺点是需要从代码层面精准地控制消费

#### Push方式

Sever端收到消息后，主动把消息推送给Client端

代码接入非常简单，适合大部分业务场景。缺点是灵活度差



|                           | Pull                               | Push       | 备注                                                         |
| ------------------------- | ---------------------------------- | :--------- | ------------------------------------------------------------ |
| 是否需要主动拉取          | 需要主动拉取各个分区消息           | 自动       | Pull消息灵活；Push使用更简单                                 |
| 位点管理                  | 用户自行管理或主动提交给Broker管理 | Broker管理 | Pull自动管理位点，消费灵活；Push交由Broker管理               |
| Topic路由变更是否影响消费 | 否                                 | 否         | Pull需要编码实现路由感知；Push自动执行Rebalance，以适应路由变更 |





### 消息过滤

#### Tag过滤

Broker使用Hash做第一次过滤（能快速过滤大量消息），由于可能出现Hash碰撞，在客户端用字符串对比做第二次过滤（漏网之鱼）

```java
DefaultMQPushConsumer consumer = new DefaultMQConsumer("GroupName");
consumer.setNamesrvAddr("127.0.0.1:9876;127.0.0.2.9876");
//订阅 Topic 和 Tag
consumer.subscribe("TopicName","TagName");

consumer.registerMessageListener(new MessageListenerConcurrently(){
    @Override
    public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> msgs,ConsumeConcurrentlyContext context){
        for(MessageExt msg : msgs){
            System.out.println("tag=" + msg.getTags);
        }
        return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
    }
});

consumer.start();
```



#### SQL92过滤

需要在启动Broker的时候配置参数

```
enableConsumeQueueExt=true;
filterSupportRetry=true;
enablePropertyFilter=true;
enableCalcFilterBitMap=true;
```

第一次过滤：使用**Bloom过滤器**的`isHit()`方法做第一次过滤

第二次过滤：执行编译后的 SQL 方法 `evaluate()` 即可过滤出最终结果

```java
DefaultMQPushConsumer consumer = new DefaultMQConsumer("GroupName");
consumer.setNamesrvAddr("127.0.0.1:9876;127.0.0.2.9876");
//订阅
consumer.subscribe("TopicName",MessageSelector.bySql("age IS NOT NULL and age between 0 and 3"));

consumer.registerMessageListener(new MessageListenerConcurrently(){
    @Override
    public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> msgs,ConsumeConcurrentlyContext context){
        for(MessageExt msg : msgs){
            System.out.println("tag=" + msg.getTags);
        }
        return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
    }
});

consumer.start();
```



#### Filter Server过滤

需要在启动Broker的时候配置参数

```
//这样可以启动一个过多个过滤服务器
filterServerNums = 大于0的数字
```

**每个过滤服务器在启动时会自动注册到NameSrv中**

第一步：用户消费者从NameSrv获取Topic路由信息，同时上传自定义的过滤器实现类源代码到Filter Server中，Filter Server 编译并实例化过滤器类

第二步：用户发送拉取消息请求到Filter Server，Filter Server通过pull consumer 从Broker拉取消息，执行过滤器中的过滤方法，返回过滤后的消息

```java
DefaultMQPushConsumer consumer = new DefaultMQConsumer("GroupName");
consumer.setNamesrvAddr("127.0.0.1:9876;127.0.0.2.9876");

ClassLoader classLoader = Thread.currentThread().getContextClassLoader();
File classFile = new File(classLoader.getResource("MessageFilterImpl.java").getFile());
//过滤器源代码
String filterCode = MixAll.file2String(classFile);

//订阅                           自定义过滤器实现类的全路径
consumer.subscribe("TopicName","org.apache.rocketmq.filter.MessageFilterImpl",filterCode);

consumer.registerMessageListener(new MessageListenerConcurrently(){
    @Override
    public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> msgs,ConsumeConcurrentlyContext context){
        for(MessageExt msg : msgs){
            System.out.println("tag=" + msg.getTags);
        }
        return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
    }
});

consumer.start();
```



## 五、NameSrv

临时保存、管理 **Topic 路由信息**（决定Topic消息发送到哪些Broker，消费者从哪些Broker消费消息）

NameSrv 节点是 **无状态** 的，即每两个 Namesrv 节点之间不通信，互相不知道对方存在。**依靠心跳保持数据一致**

在Broker、Producer、Consumer 启动时，**轮询** 配置的全部 Namesrv 节点，拉取路由信息

Broker的 **默认存活时间** 为 **120s**



### 路由数据结构

通过 ` RouteInfoManager `类实现

```java
public class RouteInfoManager {
    private static final InternalLogger log = InternalLoggerFactory.getLogger(LoggerName.NAMESRV_LOGGER_NAME);
    private final static long BROKER_CHANNEL_EXPIRED_TIME = 1000 * 60 * 2; //Broker 存活时间，默认为 120s
    private final ReadWriteLock lock = new ReentrantReadWriteLock();
    private final HashMap<String/* topic */, List<QueueData>> topicQueueTable;
    private final HashMap<String/* brokerName */, BrokerData> brokerAddrTable;
    private final HashMap<String/* clusterName */, Set<String/* brokerName */>> clusterAddrTable;
    private final HashMap<String/* brokerAddr */, BrokerLiveInfo> brokerLiveTable;
    private final HashMap<String/* brokerAddr */, List<String>/* Filter Server */> filterServerTable;

    public RouteInfoManager() {
        this.topicQueueTable = new HashMap<String, List<QueueData>>(1024);
        this.brokerAddrTable = new HashMap<String, BrokerData>(128);
        this.clusterAddrTable = new HashMap<String, Set<String>>(32);
        this.brokerLiveTable = new HashMap<String, BrokerLiveInfo>(256);
        this.filterServerTable = new HashMap<String, List<String>>(256);
    }
    ...
}
```

- **topicQueueTable**

  保存Topic和它所有Queue的信息，包括该Queue位于哪个Broker

- **brokerAddrTable**

  保存 BrokerName 与 Broker 信息的对应关系

- **clusterAddrTable**

  集群和 Broker的对应关系

- **brokerLiveTable**

  当前在线的Broker地址和Broker信息的对应关系

- **filterServerTable**

  过滤服务器信息



### **Broker定时心跳**

心跳时 Broker 将 Topic 信息和其他信息发送到 Namesrv（通过 `RequestCode.REGISTER_BROKER` 接口将心跳中的Broker信息和Topic信息存储在Namesrv中）

心跳时间为 **30s**



## 六、Broker

处理各种 TCP 请求和存储消息



### 存储目录结构

#### **CommitLog**

目录，其中包含具体的commitlog文件，每个文件大小都是1GB，可通过 mapedFileSizeCommitLog 进行配置

#### **ConsumeQueue**

目录，包含该Broker上所有的Topic对应的消费队列文件信息。消费队列文件的格式为 `“./consumequeue/Topic名字/queue id/具体消费队列文件”`

每个消费队列其实是 commitlog 的一个 **索引** ，提供给消费者做拉取消息、更新位点使用 

#### **Index**

目录，全部的文件都是按照消息key创建的hash索引。文件名是用创建时的时间戳命名的

#### **Config**

目录，保存了当前Broker中全部的Topic、订阅关系和消费进度。会定期持久化到磁盘中

#### **abort**

Broker是否异常关闭的标志。正常关闭该文件会被删除

#### **checkpoint**

Broker最近一次正常运行时的状态，比如最后一次正常刷盘的时间、最后一次正确索引的时间



### 消息存储流程

**1. Broker 接受客户端发送消息的请求并做预处理**

 

**2. Broker存储前预处理消息**



**3. 执行 `DefaultMessageStore.putMessage() `方法进行消息校验和存储模块检查**



**4. 执行 `CommitLog.putMessage()` 方法，将消息写入 CommitLog**