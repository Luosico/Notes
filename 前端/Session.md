# Session

**session 是客户端与服务器通讯会话跟踪技术，服务器与客户端保持整个通讯的会话基本信息，用来鉴别用户的身份**

session存在于服务器中，可以存储在内存中，也可以持久化到file，数据库，memcache，redis等

初次访问服务器（Tomcat），服务器会为每个用户生成 JSESSIONID（调用`request.getSession`，后面我们也可以通过这个方法来获得**session对象**）,用来标识每个用户，并通知客户端设置这个值的cookie。当用户再次访问时发送的cookie带上JSESSIONID就不需要用户手动验证而是服务器自动验证直接进入网页（即将相关信息存在session中，通过JSESSIONID的标识从服务器内存中取出需要的数据，从而完成验证操作）



## 生命周期

### 生效

在用户访问第一次访问服务器时创建，需要注意只有访问JSP、Servlet等程序时才会创建Session，只访问HTML、image等静态资源并不会创建Session,可调用request.getSession(true)强制生成Session



### 失效

Tomcat中Session的 **默认失效时间**为 **20分钟**，失效时间从session不活动开始计算时间，一旦session被访问，计时清0

1、调用Session的**invalidate**方法，主动令session失效

```java
Session session = request.getSession();
//注销该request的所有session
session.invalidate();
```

2、**设置失效时间**

- web.xml

  ```xml
  <session-config>
      <session-timeout>30</session-timeout>
  </session-config>
  ```

- 程序中设置

  ```Java
  session.setMaxInactiveInterval(30 * 60);//设置单位为秒，设置为-1永不过期
  
  //永不过期
  request.getSession().setMaxInactiveInterval(-1);
  ```

- Tomcat 的 server.xml 的 context 中

  ```xml
  <Context path="/livsorder" docBase="/home/httpd/html/livsorder" 　　defaultSessionTimeOut="3600" 
  	isWARExpanded="true" 　isWARValidated="false" 	isInvokerEnabled="true" 　　
     	isWorkDirPersistent="false"/>
  ```

  



## 分布式下的session共享

即 **分布式下的session一致性问题**，就是服务器集群之间的session共享问题

### 问题

​	假设第一次访问服务A生成一个sessionid并且存入cookie中，第二次却访问服务B客户端会在cookie中读取sessionid加入到请求头中，如果在服务B通过sessionid没有找到对应的数据那么它创建一个新的并且将sessionid返回给客户端,这样并不能共享我们的Session无法达到我们想要的目的



### 解决方案

- 使用cookie来完成（很明显这种不安全的操作并不可靠）
- 使用Nginx中的ip绑定策略，同一个ip只能在指定的同一个机器访问（不支持负载均衡）
- 利用数据库（Redis、memcached、MySQL）同步session（效率不高）
- 使用tomcat内置的session同步（同步可能会产生延迟）
- 使用token代替session
- 我们使用 **spring-session** 以及集成好的解决方案，存放在redis中