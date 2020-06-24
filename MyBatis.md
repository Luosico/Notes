# 											Mybatis

## SpringBoot部署

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