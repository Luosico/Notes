# Spring Boot

### resources文件夹

- static：存放静态文件，如css、javascript、图片；若放入html，可直接访问，如test.html，可通过localhost/test.html访问

- templates：存放动态文件，不能直接访问，要通过视图解析器访问，一般是Thymeleaf

  

### 热部署

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



### Spring事务

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



 ### Bean的作用域

  - **默认情况下，Spring应用上下文中所有bean都是单例模式（singleton）**

  - **作用域  通过 `@Scope`指定** 

      - 单例`Singleton` 在整个应用中，只创建bean的一个实例
      - 原型`Prototype` 每次注入或通过Spring应用上下文获取的时候，都会创建一个新的bean实例
      - 会话`Session` 在Web应用中，为每个会话创建一个bean实例
      - 请求`Request` 在Web应用中，为每个请求创建一个bean实例

​    

### 消息转换器（`HttpMessageConverter`）

  - `HttpMessageConverter`是spring的一个重要接口，负责将请求消息转换为一个对象（类型为T），将对象（类型为T）输出为响应消息
  - `DispatcherServlet`默认安装了`RequestMappingHandlerAdapter`作为`HandlerAdapter`的组件实现类，`HttpMessageConverter`即由`RequestMappingHandlerAdapter`使用，实现消息转换

  

### 控制器方法的数据绑定 (`DataBinder`)

    **Spring MVC通过反射机制对目标处理方法的签名进行分析，将请求消息绑定到处理方法的入参中**

  1. Spring MVC将`ServletRequest`对象及处理方法的入参对象实例传递给`DataBinder`
  2. `DataBinder`调用装配在Spring Web上下文中的`ConversionService`组件进行数据类型转换、数据格式化等工作，将`ServletRequest`中的消息填充到入参对象中
  3. `DataBinder`调用`Validator`组件对已经绑定了请求消息数据的入参对象进行数据合法性校验，最终生成数据绑定接结果`BindingResult`对象
  4. `BindingResult`包含了已完成数据绑定的入参对象，还包含相应的校验错误对象。Spring MVC抽取BindingResult中的入参对象及校验错误对象，将它们赋给处理方法的相应入参

  

 ### 控制器方法参数

  ​		**提交的表单数据是通过name获取而不是id**

  - @PathVariable    路径变量，绑定URL路径的变量

  - @RequestParam    绑定入参，当不存在是会抛出异常，若设置`required=false`,不会抛出异常

  - @RequestBody    获取请求体里的JSON数据，即`POST`提交的，`ContentType=application/json`

  - @RequestPart     获取复杂数据类型数据，一般为byte[]数组，spring提供了MultipartFile接口，可通过Part或MultipartFile获取

  - @ModelAttribute    该对象会被放到数据模型中，用于从Model、Form或URL请求参数中获取属性值

  - @SessionAttributes     将模型中的某个属性暂存到HttpSession中，以便多个请求之间可以共享这个属性

  - @CookieValue     获取请求携带的cookie值，绑定到方法的入参中

  - @RequestHeader   可以将报文头属性值绑定到方法的入参中

  - Map/Model/ModelMap    当处理器方法返回时，其中的数据会自动添加到模型中

  - 直接使用的参数，如int、String   获取路径中？后的参数

  - 直接使用的参数，获取Bean对象，如：User，里面的名称必须一致，常见于表单提交的数据   

  - Servlet API     Spring MVC会自动将Web层对应的Servlet对象传递给处理方法的入参

    - HttpServletRequest 
    - HttpServletResponse
    - HttpEntity     整个HTTP报文，也可以作为返回值
    - HttpSession

  - I/O对象作为入参    Spring MVC自动获取ServletRequest或ServletResponse的流

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

