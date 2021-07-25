# 跨域 - CORS



## 一、什么是跨域

浏览器处于安全考虑，使用了同源策略，协议、域名、端口完全一致才允许访问。跨域就是将在同源策略下不能访问的资源变成能够访问。



## 二、解决方案



### 2.1 前端解决方案

以下资源引用的标签不受同源策略的限制：

- script标签
- link标签
- img标签
- iframe标签



其他的还有jsonp



### 2.2 使用代理

- nginx或haproxy代理跨域
- nodejs中间件代理跨域



### 2.3 CORS

跨资源共享（CORS）：通过修改Http协议header的方式，实现跨域。

说的简单点就是，通过设置HTTP的响应头信息，告知浏览器哪些情况在不符合同源策略的条件下也可以跨域访问，浏览器通过解析Http协议中的Header执行具体判断。具体的Header如下：



**CROS跨域常用header**

- Access-Control-Allow-Origin: 允许哪些ip或域名可以跨域访问
- Access-Control-Max-Age: 表示在多少秒之内不需要重复校验该请求的跨域访问权限
- Access-Control-Allow-Methods: 表示允许跨域请求的HTTP方法，如：GET,POST,PUT,DELETE
- Access-Control-Allow-Headers: 表示访问请求中允许携带哪些Header信息，如：`Accept`、`Accept-Language`、`Content-Language`、`Content-Type`



## 三、Spring实现CORS



### 3.1 使用CorsFilter进行全局跨域配置

```java
    @Configuration
    public class GlobalCorsConfig {
        @Bean
        public CorsFilter corsFilter() {
    
            CorsConfiguration config = new CorsConfiguration();
            //开放哪些ip、端口、域名的访问权限，星号表示开放所有域
            config.addAllowedOrigin("*");
            //是否允许发送Cookie信息
            config.setAllowCredentials(true);
            //开放哪些Http方法，允许跨域访问
            config.addAllowedMethod("GET","POST", "PUT", "DELETE");
            //允许HTTP请求中的携带哪些Header信息
            config.addAllowedHeader("*");
            //暴露哪些头部信息（因为跨域访问默认不能获取全部头部信息）
            config.addExposedHeader("*");
    
            //添加映射路径，“/**”表示对所有的路径实行全局跨域访问权限的设置
            UrlBasedCorsConfigurationSource configSource = new UrlBasedCorsConfigurationSource();
            configSource.registerCorsConfiguration("/**", config);
    
            return new CorsFilter(configSource);
        }
    }
```



### 3.2 重写WebMvcConfigurer的addCorsMappings方法（全局跨域配置）

```java
    @Configuration
    public class GlobalCorsConfig {
        @Bean
        public WebMvcConfigurer corsConfigurer() {
            return new WebMvcConfigurer() {
                @Override
                public void addCorsMappings(CorsRegistry registry) {
                    registry.addMapping("/**")    //添加映射路径，“/**”表示对所有的路径实行全局跨域访问权限的设置
                            .allowedOrigins("*")    //开放哪些ip、端口、域名的访问权限
                            .allowCredentials(true)  //是否允许发送Cookie信息 
                            .allowedMethods("GET","POST", "PUT", "DELETE")     //开放哪些Http方法，允许跨域访问
                            .allowedHeaders("*")     //允许HTTP请求中的携带哪些Header信息
                            .exposedHeaders("*");   //暴露哪些头部信息（因为跨域访问默认不能获取全部头部信息）
                }
            };
        }
    }
```



### 3.3 使用CrossOrigin注解（局部跨域配置）

- 将CrossOrigin注解加在Controller层的方法上，该方法定义的RequestMapping端点将支持跨域访问

- 将CrossOrigin注解加在Controller层的类定义处，整个类所有的方法对应的RequestMapping端点都将支持跨域访问

```java
    @RequestMapping("/cors")
    @ResponseBody
    @CrossOrigin(origins = "http://localhost:8080", maxAge = 3600) 
    public String cors( ){
        return "cors";
    }
```



### 3.4 使用HttpServletResponse设置响应头(局部跨域配置)

```java
    @RequestMapping("/cors")
    @ResponseBody
    public String cors(HttpServletResponse response){
        //使用HttpServletResponse定义HTTP请求头，最原始的方法也是最通用的方法
        response.addHeader("Access-Control-Allow-Origin", "http://localhost:8080");
        return "cors";
    }
```

