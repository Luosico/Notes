# Rabbit



## 一、示例

**生产者**

```java
public class RabbitProducer {
    public static final String EXCHANGE_NAME = "exchange_demo";
    public static final String ROUTING_KEY = "routingkey_demo";
    public static final String QUEUE_NAME = "queue_demo";
    public static final String IP_ADDRESS = "localhost";
    public static final int PORT = 5672;

    public static Connection connection;
    public static Channel channel;

    public RabbitProducer() throws IOException, TimeoutException {
        init();
    }

    private void init() throws IOException, TimeoutException {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost(IP_ADDRESS);
        factory.setPort(PORT);
        factory.setUsername("user");
        factory.setPassword("123456");

        // 创建连接
        connection = factory.newConnection();
    }

    public void produce(String message) throws IOException, TimeoutException {

        // 创建信道
        channel = connection.createChannel();
        // 创建一个 type="direct"、持久化的、非自动删除的交换器
        channel.exchangeDeclare(EXCHANGE_NAME, "direct", true, false, null);
        // 创建一个持久化、非排他的、非自动删除的队列
        channel.queueDeclare(QUEUE_NAME, true, false, false, null);
        // 通过路由键绑定队列和交换器
        channel.queueBind(QUEUE_NAME, EXCHANGE_NAME, ROUTING_KEY);
        // 发送消息
        channel.basicPublish(EXCHANGE_NAME, ROUTING_KEY, MessageProperties.TEXT_PLAIN, message.getBytes());
    }

    /**
     * 关闭资源
     */
    public void shutdown() throws IOException, TimeoutException {
        connection.close();
        channel.close();
    }
}
```



**消费者**

```java
public class RabbitConsumer {
    public static final String QUEUE_NAME = "queue_demo";
    public static final String IP_ADDRESS = "localhost";
    public static final int POST = 5672;

    public static Connection connection;
    public static Channel channel;

    public RabbitConsumer() {
        try {
            init();
        } catch (IOException | TimeoutException e) {
            e.printStackTrace();
        }
    }

    private void init() throws IOException, TimeoutException {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setUsername("user");
        factory.setPassword("123456");

        Address[] addresses = new Address[]{new Address(IP_ADDRESS)};
        connection = factory.newConnection(addresses);

        channel = connection.createChannel();
        // 设置客户端最多接受未被 ack 的消息的个数
        channel.basicQos(64);
    }

    public void consume() throws IOException {
        Consumer consumer = new DefaultConsumer(channel){
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                System.out.println(new String(body));
                try {
                    TimeUnit.SECONDS.sleep(1);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                channel.basicAck(envelope.getDeliveryTag(), false);
            }
        };

        // 进行消费，设置队列
        channel.basicConsume(QUEUE_NAME, consumer);
    }

    public void shutdown() throws IOException, TimeoutException {
        connection.close();
        channel.close();
    }
}
```

**生产者和消费者都可以声明交换器和队列，在声明时，若对应名字的交换器和队列已存在，只要声明的参数完全匹配现存的交换器或队列，直接成功返回；如果不匹配，抛出异常**



## 二、重要概念

### 1、交换器 Exchange

​	生产者将消息发送到 Exchange，由交换器将消息路由到一个或多个队列中。如果路由不到，或许会返回给生产者，或许直接丢掉。



### 2、路由键 RoutingKey

​	生产者将消息发送到交换器的路由规则，RoutingKey需要和绑定建 BingdingKey 联合使用。



### 3、绑定 Bingding

​	通过绑定将交换器与队列关联起来，在绑定的时候回指定绑定建 BingdingKey



## 三、交换器的类型

- **fanout**

    把所有发送到该交换器的消息路由到所有与该交换器绑定的队列中

- **direct**

    把消息路由到BindingKey和RoutingKey完全匹配的队列中

- **top**

    支持BindingKey和RoutingKey的模糊匹配，*匹配一个单词，#匹配多规格单词（可以是零个）、

- **headers**

    根据发送的消息内容中的headers属性进行匹配



## 四、常用方法

### exchangeDeclare

```java
DeclareOk exchangeDeclare(String exchange, String type, boolean durable, boolean autoDelete, Map<String, Object> arguments)
```

- exchange: 交换器名称
- type: 交换器类型
- durable: 是否持久化
- autoDelete: 是否自动删除。自动删除的前提是至少有一个队列或者交换器与这个交换器绑定，之后所有与这个交换器绑定的队列或者交换器都与此解绑，而不是“此交换器连接的客户端都断开时，RabbitMQ自动删除本交换器”
- internal：是否是内置的交换器。生产者无法发送消息到内置的交换器，只能通过交换器到交换器这种形式
- argument：其他一些结构化参数。



### queueDeclare

生产者和消费者都可以使用queueDeclare来声明一个队列，但如果消费者在同一信道上订阅了另一个队列，就无法再声明队列了。必须先取消订阅，并将信道置为“传输”模式，之后才能声明队列。

```java
# 默认创建一个有RabbitMQ命名的队列（也称匿名队列）、排他的、自动删除的、非持久化的队列
DeclareOk queueDeclare()
DeclareOk queueDeclare(String queue, boolean durable, boolean exclusive, boolean autoDelete, Map<String, Object> arguments)
```

- queue：队列名称
- durable：是否持久化
- exclusive：是否排他。排他队列仅对首次声明它的**连接**可见，并在连接断开时自动删除
- autoDelete：是否自动删除。自动删除的前提是至少有一个消费者连接到这个队列，之后所有与这个队列连接的消费者都断开时，才会自动删除。
- arguments：参数



### queueBind

**将队列和交换器绑定**

```java
BindOk queueBind(String queue, String exchange, String routingKey);
BindOk queueBind(String queue, String exchange, String routingKey, Map<String, Object> arguments);
```

- queue：队列名称
- exchange：交换器名称
- routingKey：路由键
- arguments：参数

**将队列和交换器解绑**

```java
UnbindOk queueUnbind(String queue, String exchange, String routingKey);
UnbindOk queueUnbind(String queue, String exchange, String routingKey, Map<String, Object> arguments);
```



### exchangeBind

**交换器与交换器绑定**

```java
BindOk exchangeBind(String destination, String source, String routingKey);
BindOk exchangeBind(String destination, String source, String routingKey, Map<String, Object> arguments);
void exchangeBindNoWait(String destination, String source, String routingKey, Map<String, Object> arguments);
UnbindOk exchangeUnbind(String destination, String source, String routingKey);
UnbindOk exchangeUnbind(String destination, String source, String routingKey, Map<String, Object> arguments);
void exchangeUnbindNoWait(String destination, String source, String routingKey, Map<String, Object> arguments);
```





## 五、发送消息

主要为 Channel.basicPublish

```java
void basicPublish(String exchange, String routingKey, BasicProperties props, byte[] body);
void basicPublish(String exchange, String routingKey, boolean mandatory, BasicProperties props, byte[] body);
void basicPublish(String exchange, String routingKey, boolean mandatory, boolean immediate, BasicProperties props, byte[] body);
```

**普通发送**

```java
channel.basicPublish(exchangeName, routingKey, mandatory, MessageProperties.PERSISTENT_TEXT_PLAIN, messageBodyBytes);
```



**设定消息属性**

```java
channel.basicPublish(exchangeName, routingKey, new AMQP.BasicProperties.Builder()
               .contentType("text/plain")
               .deliveryMode(2)
               .priority(1)
               .userId("hidden")
               .build(),
               messageBodyBytes);
```



**带 headers  的消息**

```java
Map<String, Object> headers = new HashMap<>();
headers.put("location", "here");
headers.put("time", "today");
channel.basicPublish(exchangeName, routingKey,
                    new AMQP.BasicProperties.Builder()
                    .headers(headers)
                    .build(),
                    messageBodyBytes);
```



**带过期时间**

```java
channel.basicPublish(exchangeName, routingKey,
                    new AMQP.BasicProperties.Builder()
                    .expiration("60000")
                    .build(),
                    messageBodyBytes);
```





## 六、消费消息

**RabbbitMQ的消费模式分为：推（push）模式和拉（pull）模式。**push采用 `Basic.Consumer`，pull采用`Basic.Get`

### 推模式

```java
String basicConsume(String queue, Consumer callback);
String basicConsume(String queue, DeliverCallback deliverCallback, CancelCallback cancelCallback);
String basicConsume(String queue, DeliverCallback deliverCallback, ConsumerShutdownSignalCallback shutdownSignalCallback);
String basicConsume(String queue, DeliverCallback deliverCallback, CancelCallback cancelCallback, ConsumerShutdownSignalCallback shutdownSignalCallback);
String basicConsume(String queue, boolean autoAck, Consumer callback);
String basicConsume(String queue, boolean autoAck, String consumerTag, Consumer callback);
String basicConsume(String queue, boolean autoAck, DeliverCallback deliverCallback, CancelCallback cancelCallback);
...
```

- queue：队列名称
- autoAck：是否自动确认。建议设为false，显示确认可以防止消息不必要地丢失，通过channel.basicAck确认
- consumerTag：消费者标签，用来区分多个消费者
- noLocal：设置为true，则表示不能将同一个Connection中生产者发送的消息传送给这个Connection中的消费者
- exclusive：是否排他
- arguments：消费者的其他参数
- callback：消费者的回调函数，例如DefaultComsumer



**消费消息一般通过实现 Consumer 接口或者继承 DefaultConsumer 类来实现**



### 拉模式

```java
GetResponse basicGet(String queue, boolean autoAck);
```



### 确认与拒绝

**确认消息**

RabbitMQ通过ack参数，提供了消息确认机制。

当ack为true时，RabbitMQ会自动把发送出去的消息置为确认，然后从内存（或者磁盘）中删除，而不管消费者是否真正消费到了这些消息

当ack为false时，RabbitMQ会等待消费者显示地回复确认信号才从内存（或磁盘）中移去消息（实质上是先打上删除标记，之后再删除）

```java
void channel.basicAck(long deliveryTag, boolean multiple);
```

RabbitMQ 不会为未确认的消息设置过期时间，这允许消费者消费一条消息的时间可以很久很久。

判断这条消息是否需要重新投递给消费者的唯一依据是消费该消息的消费者连接是否已经断开。



**拒绝消息**

```java
void channel.basicReject(long deliveryTag, boolean requeue);
```

当requeue设置为true，RabbitMQ会重新将这条消息存入队列；设置为false，则立即移除



**批量拒绝消息**

```java
void basicNack(long deliveryTag, boolean multiple, boolean requeue);
```

当multiple为false时，只拒绝编号为deliveryTage 的这一条消息；为true则表示拒绝 deliveryTag 编号之前所有未被当前消费者确认的消息



**重新发送消息**

```java
RecoverOk basicRecover();
RecoverOk basicRecover(boolean requeue);
```

**RabbitMQ重新发送还未被确认的消息**

**requeue 默认为true**

requeue 为 true，未被确认的消息会被重新加入到队列中，可能会被分配到与之前不同的消费者。

requeue 为 false，同一条消息会被分配给与之前相同的消费者



### 死信队列

basicReject 和 basicNeck 中 requeue 设置为false，相当于启用了 “**死信队列**”功能。

死信队列可以通过检测被拒绝或者未送达的消息来追踪问题