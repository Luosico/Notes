# Sentenel

Sentenel是面向分布式服务架构的轻量级流量控制组件，主要以流量为切入点，从限流、流量整形、服务降级、系统负载保护等多个维度来保障微服务的稳定性。



### 实现步骤

1. 定义资源
2. 定义限流规则
3. 检验规则是否生效



### 保护规则

- 流量控制规则
- 熔断降级规则
- 系统保护规则
- 来源访问控制规则
- 热点参数规则

### 流量控制规则

```java
//限流规则
FlowRUle rule = new FlowRule();
//设置需要保护的资源
rule.setResource("resource");
//限流阈值
rule.setCount(20);
//限流阈值类型
rule.setGrade(RuleConstant.FLOW_GRADE_QPS);
//限流调用来源
rule.setLimitApp("default");
//调用关系限流策略——直接、链路、关联
rule.setStrategy(RuleConstant.STRATETY_CHAIN);
//流量控制行为
rule.setControlBehavior(RuleConstant.CONTROL_BEHAVIOR_DEFAULT);
//集群限流
rule.setClusterMode(false);
```



**流量统计有两种类型，通过grade属性来控制**：

- 并发线程数（FLOW_GRADE_THREAD）
- QPS（FLOW_GRADE_QPS）



**QPS流量控制行为**

当QPS超过阈值，就会触发流量控制行为，这种行为是通过`controlBehavior`来设置

- 直接拒绝（RuleConstant.CONTROL_BEHAVIOR_DEFAULT） **默认**，超出阈值时抛出FlowException
- 冷启动（RuleConstant.CONTROL_BEHAVIOR_WARM_UP），也叫预热
- 匀速排队（RuleConstant.CONTROL_BEHAVIOR_RATE_LIMITER），通过漏桶限流算法实现
- 冷启动+匀速排队（RuleConstant.CONTROL_BEHAVIOR_WARM_UP_RATE_LIMITER）



**调用关系流量策略**

调用关系分为调用方和被调用方，一个方法又可能会调用其他方法，形成一个调用链。所谓的调用关系流量策略，就是根据不同的调用维度来触发流量控制

- 调用方

    设置`limitApp`来设置来源信息，有三个选项

    - default：不区分调用者，也就是任何访问调用者的请求都会进行限流统计
    - {some_origin_name}：设置特定的调用者，只对这个调用者的请求进行流量统计和控制
    - other：表示对{some_origin_name}外的其他调用者进行流量统计

- 调用链路入口

- 具有关系的资源流量控制（关联流量控制）



### 服务熔断

```java
DegradeRule rule = new DegradeRule();
rule.setResource("resource");
rule.setCount(10);
//熔断策略
rule.setGrade(RuleConstant.DEGRADE_GRADE_RT);
//熔断降级的时间窗口
rule.setTimeWindown(10);
//RT模式下1秒内多少个请求的平均RT超出阈值后触发熔断，默认值为5
rule.setMinRequestAmount(5);
//触发的异常熔断最小请求数，默认值为5
rule.setRtSlowRequestAmount(5);
```

**熔断策略**

- 平均响应时间（RuleConstant.DEGRADE_GRADE_RT）默认为5，秒
- 异常比例（RuleConstant.DEGRADE_GRADE_EXCEPTION_RATIO）默认为5，秒
- 异常数（RuleConstant.DEGRADE_GRADE_EXCEPTION_COUNT），分钟



### 配置限流资源

- Sentinel starter 默认情况下会为所有的HTTP服务提供限流埋点
- 在特定方法上通过 @SentinelResource 注解定义
- 通过SphU.entry()方法



**官方文档**

https://github.com/alibaba/spring-cloud-alibaba/wiki/Sentinel

https://github.com/alibaba/Sentinel/wiki/%E4%BB%8B%E7%BB%8D