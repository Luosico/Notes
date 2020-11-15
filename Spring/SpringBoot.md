# Spring Boot

## resources文件夹

- static：存放静态文件，如css、javascript、图片；若放入html，可直接访问，如test.html，可通过localhost/test.html访问

- templates：存放动态文件，不能直接访问，要通过视图解析器访问，一般是Thymeleaf

  

## 热部署

- 添加依赖

  ```xml
  <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-devtools</artifactId>
  	<optional>true</optional> //可选依赖，不会被其他项目继承，若要使用，必须显示声明使用
  </dependency>
  ```

- 配置idea

  在 File -> setting -> Build,Execuation,Deployment -> Compiler 中勾选 Builder project automatically



## Spring事务

- **隔离级别**
  - isolation_default								同数据库事务隔离级别
  - isolation_read_uncommited            未提交读，脏数据
  - isolation_read_commited                  提交读，不可重复读
  - isolation_repeatable_read                 可重复读，幻读
  - isolation_serializable                           可串行化
- **传播行为**
  - required                当前没有，就新建事务，若已存在事务，加入这个事务
  - supports                支持当前事务。如果当前没有事务，就以非事务方式运行
  - mandatory            使用当前事务。如果当前没有事务，就抛出异常
  - requires_new        新建事务。如果当前存在事务，就把当前事务挂起
  - not_supported     以非事务方式执行。若果当前存在事务，就把当前事务挂起
  - never                     以非事务方式执行。若果存在当前事务，抛出异常
  - nested                   

​    

## 消息转换器（`HttpMessageConverter`）

  - `HttpMessageConverter`是spring的一个重要接口，负责将请求消息转换为一个对象（类型为T），将对象（类型为T）输出为响应消息
  - `DispatcherServlet`默认安装了`RequestMappingHandlerAdapter`作为`HandlerAdapter`的组件实现类，`HttpMessageConverter`即由`RequestMappingHandlerAdapter`使用，实现消息转换

  

## 控制器方法的数据绑定 (`DataBinder`)

**Spring MVC通过反射机制对目标处理方法的签名进行分析，将请求消息绑定到处理方法的入参中**

  1. Spring MVC将`ServletRequest`对象及处理方法的入参对象实例传递给`DataBinder`
  2. `DataBinder`调用装配在Spring Web上下文中的`ConversionService`组件进行数据类型转换、数据格式化等工作，将`ServletRequest`中的消息填充到入参对象中
  3. `DataBinder`调用`Validator`组件对已经绑定了请求消息数据的入参对象进行数据合法性校验，最终生成数据绑定接结果`BindingResult`对象
  4. `BindingResult`包含了已完成数据绑定的入参对象，还包含相应的校验错误对象。Spring MVC抽取BindingResult中的入参对象及校验错误对象，将它们赋给处理方法的相应入参

  

 ## 控制器方法参数

**提交的表单数据是通过 name 获取而不是 id**

  - **@PathVariable**    路径变量，绑定URL路径的变量

  - **@RequestParam**    绑定入参，当不存在是会抛出异常，若设置`required=false`,不会抛出异常

  - **@RequestBody**    获取请求体里的JSON数据，即`POST`提交的，`ContentType=application/json`

  - **@RequestPart**     获取复杂数据类型数据，一般为byte[]数组，spring提供了MultipartFile接口，可通过Part或MultipartFile获取

  - **@ModelAttribute**    该对象会被放到数据模型中，用于从Model、Form或URL请求参数中获取属性值

  - **@SessionAttributes**     将模型中的某个属性暂存到HttpSession中，以便多个请求之间可以共享这个属性

  - **@CookieValue**     获取请求携带的cookie值，绑定到方法的入参中

  - **@RequestHeader**   可以将报文头属性值绑定到方法的入参中

  - **Map/Model/ModelMap**    当处理器方法返回时，其中的数据会自动添加到模型中

  - **直接使用的参数**，如int、String   获取路径中？后的参数

  - **直接使用的参数**，获取Bean对象，如：User，里面的名称必须一致，常见于表单提交的数据   

  - **Servlet API**     Spring MVC会自动将Web层对应的Servlet对象传递给处理方法的入参

    - HttpServletRequest 
    - HttpServletResponse
    - HttpEntity     整个HTTP报文，也可以作为返回值
    - HttpSession

  - **I/O对象**作为入参    Spring MVC自动获取ServletRequest或ServletResponse的流

    - InputStream/OutputStream
    - Reader/Writer

    

- ### 控制器返回值

  - ModelAndView     通过该对象添加模型数据
  - String                     返回一个视图名



- ### 返回json
  
  - json字符串：将对象转换成String类型
  - json对象（消息转化器会自动处理）
    - Map 对象
    - domain 对象



## 拦截器和过滤器的区别

- **过滤器**：依赖于servlet容器。在实现上基于函数回调，可以对几乎所有请求进行过滤，但是缺点是一个过滤器实例只能在容器初始化时调用一次。使用过滤器的目的是用来做一些过滤操作，获取我们想要获取的数据，比如：在过滤器中修改字符编码；在过滤器中修改HttpServletRequest的一些参数，包括：过滤低俗文字、危险字符等
- **拦截器**：依赖于web框架，在SpringMVC中就是依赖于SpringMVC框架。在实现上基于 Java的反射机制，属于面向切面编程（AOP）的一种运用。由于拦截器是基于web框架的调用，因此可以使用Spring的依赖注入（DI）进行一些业务操作，同时一个拦截器实例在一个controller生命周期之内可以多次调用。但是缺点是只能对controller请求进行拦截，对其他的一些比如直接访问静态资源的请求则没办法进行拦截处理

|                | filter                                     | Interceptor                                           |
| -------------- | ------------------------------------------ | ----------------------------------------------------- |
| 多个的执行顺序 | 根据filter mapping配置的先后顺序           | 按照配置的顺序，但是可以通过order控制顺序             |
| 规范           | 在Servlet规范中定义的，是Servlet容器支持的 | Spring容器内的，是Spring框架支持的。                  |
| 使用范围       | 只能用于Web程序中                          | 既可以用于Web程序，也可以用于Application、Swing程序中 |
| 深度           | Filter在只在Servlet前后起作用              | 拦截器能够深入到方法前后、异常抛出前后等              |

**过滤器**

```java
@WebFilter(urlPatterns = "/*", filterName = "logFilter")
public class LogCostFilter implements Filter {
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
 
    }
 
    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        long start = System.currentTimeMillis();
        filterChain.doFilter(servletRequest, servletResponse);
        System.out.println("LogFilter2 Execute cost=" + (System.currentTimeMillis() - start));
    }
 
    @Override
    public void destroy() {
 
    }
}

```

**拦截器**

```java
public class LogCostInterceptor implements HandlerInterceptor {
    long start = System.currentTimeMillis();
    @Override //请求前
    public boolean preHandle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o) throws Exception {
        start = System.currentTimeMillis();
        return true;
    }
 
    @Override //请求后
    public void postHandle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, ModelAndView modelAndView) throws Exception {
        System.out.println("Interceptor cost="+(System.currentTimeMillis()-start));
    }
 
    @Override
    public void afterCompletion(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, Exception e) throws Exception {
    }
}

//进行配置
@Configuration
public class InterceptorConfig extends WebMvcConfigurerAdapter {
 
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new LogCostInterceptor()).addPathPatterns("/**");
        super.addInterceptors(registry);
    }
}
```



#### 区别

- 拦截器是基于java的 **反射** 机制的，而过滤器是基于 **函数回调**
- 拦截器不依赖与servlet容器，过滤器依赖与servlet容器
- 拦截器只能对action请求起作用，而过滤器则可以对几乎所有的请求起作用
- 拦截器可以访问action上下文、值栈里的对象，而过滤器不能访问
- 在action的生命周期中，拦截器可以多次被调用，而过滤器只能在容器初始化时被调用一次
- 拦截器可以获取IOC容器中的各个bean，而过滤器就不行，这点很重要，在拦截器里注入一个service，可以调用业务逻辑



## IoC

### 1、控制反转（IoC）、依赖倒置、依赖注入（DI） 

**控制反转是一种设计模式，其遵循软件工程中的依赖倒置原则，依赖注入是Spring实现控制反转的一种方式**

### 2、原理解析

#### (1) BeanFactory

整个 IoC容器最顶层接口， 也是容器，是一个低配版的IoC容器，定义了IoC容器的基本功能

#### (2) ApplicationContext

顶级父类是 **BeanFactory** 接口，是高配版的IoC容器

#### (3) BeanDefinition

 	抽象了对 **JavaBean** 的定义，管理各种 JavaBean 及 JavaBean 相互之间的依赖关系，是容器实现依赖反转的核心数据结构，

Spring 将开发人员定义的 JavaBean 的数据结构转化为内存中的 BeanDefinition 数据结构进行维护，**是Bean在内存中的状态**

#### (4) BeanPostProcessor

​	Bean的后置处理器，是一个监听器，可以监听容器触发的事件。将它向IOC容器注册后，容器中管理的Bean具备了接收IOC容器事件回调的能力。BeanPostProcessor是一个接口类，有两个接口方法，postProcessBeforeInitialization提供Bean初始化前的回调入口；postProcessAfterInitialization 提供Bean初始化后的回调入口`AbstractAutowireCapableBeanFactory#initializeBean`

### 3、启动流程

[![wDGqRx.md.jpg](https://s1.ax1x.com/2020/09/14/wDGqRx.md.jpg)](https://imgchr.com/i/wDGqRx)



### 4、IoC容器的实现

​	Spring提供了各种各样的容器，有DefaultListableBeanFactory、FileSystemXmlApplicationContext等，这些容器都是基于BeanFactory，BeanFactory实现了容器的基础功能，包括containsBean能够判断容器是否含有指定名称的Bean，getBean获取指定名称参数的Bean等。

Spring通过`refresh()`方法对容器进行初始化和资源的载入

首先通过ResourceLoader的Resource接口定位到存储Bean信息的路径

​	第二个过程是BeanDefinition载入，把定义好的Bean表示成IOC容器的内部数据结构BeanDefinition，通过定义BeanDefinition来管理应用的各种对象及依赖关系，其是容器实现依赖反转功能的核心数据结构

​	第三个过程是BeanDefinition注册，容器解析得到BeanDefinition后，需要在容器中注册，这由IOC实现BeanDefinitionRegistry接口来实现，注册过程是IOC容器内部维护了一个ConcurrentHasmap来保存得到的BeanDefinition。如果某些Bean设置了lazyinit属性，Bean的依赖注入会在这个过程预先完成，而不需要等到第一次使用Bean的时候才触发。



### 5、Spring DI(依赖注入)的实现

**Spring 的依赖注入发生在以下两种情况：**

1. 用户第一次调用`getBean()`方法

2. bean配置了lazy-init=false，容器会在解析注册Bean定义的时候进行预实例化，触发依赖注入

**getBean()**方法定义在BeanFactory接口中，具体实现在子类AbstractBeanFactory中，过程如下：

1. getBean()方法最终是委托给doGetBean方法来实例化Bean，doGetBean方法会先从缓存中找是否有创建过，没有再从父工厂中去查找

2. 如果父工厂中没有找到，会根据Bean定义的模式来创建Bean，单例模式的Bean会先从缓存中查找，确保只创建一次，原型模式的Bean每次都会创建，其他模式根据配置的不同生命周期来选择合适的方法创建。创建的具体方法通过匿名类中getObject,并委托给createBean来完成bean的实例化。

3. 在createBean中，先对Bean进行一些准备工作，然后会应用配置的前后处理器，如果创建成功就直接返回该代理Bean

4. 没有创建代理Bean的话，会创建指定的Bean实例，委托给doCreateBean完成，该过程会通过提前实例化依赖Bean，并写入缓存来解决Bean的循环依赖

5. 通过populateBean注入Bean属性，并调用init-method初始化方法

6. 注册实例化的Bean



### 6、Bean的循环依赖

​	**比如A > B , B > C , C > A**，可以分为

1. 构造器的循环依赖**（无法解决）**
2. field属性的循环依赖

​	 <font size='5'>可通过**三级缓存**解决</font>，分为

- **singletonObjects  	           单例对象的Cache**  

- **earlySingletonObjects       提前曝光的单例对象的Cache**

- **singletonFactories              单例对象工厂的Cache**

  ​	A首先完成了初始化的第一步，并且将自己提前曝光到singletonFactories中，此时进行初始化的第二步，发现自己依赖对象B，此时就尝试去get(B)，发现B还没有被create，所以走create流程，B在初始化第一步的时候发现自己依赖了对象A，于是尝试get(A)，尝试一级缓存singletonObjects(肯定没有，因为A还没初始化完全)，尝试二级缓存earlySingletonObjects（也没有），尝试三级缓存singletonFactories，由于A通过ObjectFactory将自己提前曝光了，所以B能够通过ObjectFactory.getObject拿到A对象(虽然A还没有初始化完全，但是总比没有好呀)，B拿到A对象后顺利完成了初始化阶段1、2、3，完全初始化之后将自己放入到一级缓存singletonObjects中。此时返回A中，A此时能拿到B的对象顺利完成自己的初始化阶段2、3，最终A也完成了初始化，进去了一级缓存singletonObjects中，而且更加幸运的是，由于B拿到了A的对象引用，所以B现在hold住的A对象完成了初始化。

  ​	**因为加入singletonFactories三级缓存的前提是执行了构造器，所以构造器的循环依赖没法解决。**



### 7、Bean的作用域

  - **默认情况下，Spring应用上下文中所有bean都是单例模式（singleton）**

  - **作用域  通过 `@Scope`指定** 

    - 单例`Singleton` 在整个应用中，只创建bean的一个实例
    - 原型`Prototype` 每次注入或通过Spring应用上下文获取的时候，都会创建一个新的bean实例
    - 会话`Session` 在Web应用中，为每个会话创建一个bean实例
    - 请求`Request` 在Web应用中，为每个请求创建一个bean实例
    - `globalSession` 全局的Session中
    
    

### 8、Bean的生命周期

[![whyVAK.md.jpg](https://s1.ax1x.com/2020/09/18/whyVAK.md.jpg)](https://imgchr.com/i/whyVAK)



## AOP

