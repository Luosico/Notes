# 																	Java 网络编程 
## 一、Java网络知识

* 所有现代计算机网络都是包交换（分组交换）网络
* Java不支持ICMP，也不允许发送原始IP数据报
* Java支持的协议只有TCP和UDP，以及建立在TCP和UDP之上的应用层协议
  
## 二、Internet地址
 1. 定义：连接到Internet的设备称为节点（node）。计算机节点称为主机。每个节点或主机都由至少一个唯一的数来标识，这称为Internet地址或IP地址 
 2. **InetAddress**：是Java对IP地址的高层表示。一般的讲，包括一个主机名和一个IP地址  
 	* 通过 &nbsp; `InetAdress.getByAddress()` &nbsp;来获取InetAddress对象，传入IP地址是byte[]数组，这个实际上是通过**DNS查询**进行的  
 	`byte[] address = { 107, 23, (byte)216, (byte)196 }`
 	* `InetAddress.getByName()`是仅建立InetAddress对象，不会进行DNS查询，这个地址标识的主机不一定存在或者能正确映射到IP地址，显式调用`getHostName()`获取主机名才会完成主机名的DNS查找
    * InetAddress类会缓存DNS查找的结果
    * InetAddressl对象没有`setxxx()`方法，不能改动地址，所以是**线程安全**的
    * 测试可达性：`isReachable()`方法，测试一个特定节点对当前主机是否可达，其实使用的是`traceout`,更确切的说是`ICMP echo`请求
 3. **Networkinterface**:表示一个本地IP地址，可以是一个物理接口，如额外的以太网卡（常见于防火墙和路由器），也可以是一个虚拟接口，与机器的其他IP地址绑定到同一个物理硬件

## 三、URL和URI