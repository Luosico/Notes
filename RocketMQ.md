# RocketMQ

## 消息队列

### 定义

**从广义上讲是一种消息队列服务中间件，提供一套完整的信息生产、传递、消费的软件系统，采用RPC进行通信**

### 作用

- 削峰填谷
- 程序间解耦
- 异步处理
- 数据的最终一致性



## 组件

- Producer
- Consumer
- Broker
- Namesrv



## 生产者

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





## 消费者

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

也叫消息进度持久化

分为：远程位点管理 和 本地位点管理

- LocalFileOffsetStore
- RemoteBrokerOffsetStore

支持：

- 定时持久化：通过定时任务来定时持久化
- 不定时持久化：也叫 Pull-And-Commit，在执行Pull方法的同时，把队列最新消费位点信息发送给Broker