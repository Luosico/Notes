# Mybatis

### 一、SpringBoot部署

  - 在`application.properties`添加mybatis配置、数据源

    ```
    #mybatis
    mybatis.config-location=classpath:mybatis/mybatis-config.xml
    mybatis.mapper-locations=classpath:mybatis/mappers/*.xml
    
    #数据源
    spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
    spring.datasource.url=jdbc:mysql://localhost:3306/seckill?    useUnicode=true&characterEncoding=utf8&serverTimezone=Asia/Shanghai
    spring.datasource.username=root
    spring.datasource.password=*****
    ```
    
- 创建 `mybatis-config.xml（非必须）` 和 `mapper.xml` 文件

- 创建`mapper接口`，并添加`@Repository`注解，使用可直接通过自动注入来完成sql操作

  ```
  	@Autowired
      UserMapper userMapper; 
  
      @RequestMapping(value = "/hello",method = RequestMethod.GET)
      public String hello() throws IOException {
          User user = userMapper.selectUser("admin");
          System.out.println(user.getUserName());
          return "hello";
      }
  ```

- 添加`@MapperScan("com.mapper")`才能扫描到`mapper接口`文件



### 二、`#{}` 和 `${}`

- **#{}** 经过预编译，是安全的；而 **${}** 未经过预编译，可能存在SQL注入风险
- Mybatis官方希望在进行数据库操作的时候将具体的参数作为条件的时候推荐使用#{}，因为这种方式更为安全，防止sql注入，如果使用order by 根据字段排序的这种，推荐使用${}这种方式
    - 使用#{}这种方式的时候会进行预处理并换成转义的字符串，也就是用?代替
    - 使用${}的时候，并不会进行预处理，而是直接将这个值设置进去