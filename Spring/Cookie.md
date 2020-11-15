# Cookie

cookies只能是非空白字符的ASCII文本，不能包含逗号和分号

```
HTTP/1.1 200 OK
Content-type:text/html
Set-Cookie:name=abcd
```



## 获取Cookie

```java
@GetMapping("/")
public String readCookie(@CookieValue(value = "username", defaultValue = "Atta") String username) {
    return "Hey! My username is " + username;
}
```

**可以通过`HttpServletRequest`作为Controller的方法参数来获取所有的Cookie，调用 `getCookies()` 方法**

```java
@GetMapping("/all-cookies")
public String readAllCookies(HttpServletRequest request) {

    //获取所有 Cookie
    Cookie[] cookies = request.getCookies();
    
    if (cookies != null) {
        return Arrays.stream(cookies)
                .map(c -> c.getName() + "=" + c.getValue()).collect(Collectors.joining(", "));
    }

    return "No cookies";
}
```



## 设置Cookie

```java
@GetMapping("/change-username")
public String setCookie(HttpServletResponse response) {
    // create a cookie
    Cookie cookie = new Cookie("username", "Jovan");

    //add cookie to response
    response.addCookie(cookie);

    return "Username is changed!";
}
```



## 过期时间

如果Cookie没有设置过期时间，会一直存活到session过期，这种Cookie称为 `session cookies`，他们会存活到用户关闭浏览器或清楚他们的cookies

```java
// create a cookie
Cookie cookie = new Cookie("username", "Jovan");
//设置过期时间
cookie.setMaxAge(7 * 24 * 60 * 60); // expires in 7 days

//add cookie to response
response.addCookie(cookie);
```



## Secure Cookie

安全cookie是仅通过加密的HTTPS连接发送到服务器的cookie。 无法通过未加密的HTTP连接将安全cookie传输到服务器。

```java
Cookie cookie = new Cookie("username", "Jovan");
cookie.setMaxAge(7 * 24 * 60 * 60); // expires in 7 days

//设置为安全cookie
cookie.setSecure(true);

//add cookie to response
response.addCookie(cookie);
```



## HttpOnly Cookie

HttpOnly cookie用于防止跨站点脚本（XSS）攻击，无法通过JavaScript的API访问，即无法通过js脚本读取cookie信息

当为cookie设置标志时，它告诉浏览器该特定cookie仅应由服务器访问

```java
// create a cookie
Cookie cookie = new Cookie("username", "Jovan");
cookie.setMaxAge(7 * 24 * 60 * 60); // expires in 7 days
cookie.setSecure(true);
cookie.setHttpOnly(true);

//add cookie to response
response.addCookie(cookie);
```



## Cookie作用域

包括域和路径，即**Domain**和**Path**

默认的作用域为最初的URL和所有的子目录



### Domain

**默认为当前主机，不包括子域名**

只能为直属的域设置cookie，即只能为它和它的子域设置cookie

必须加 `.`才能设置为子域名也可以使用cookie，例如`Domain=.luosico.xyz`



### Path

只能作用当前目录和子目录，不能作用于上一级的目录， `/` 代表当前站点的根目录

但可以使用cookie的Path属性改变路径

```java
// create a cookie
Cookie cookie = new Cookie("username", "Jovan");
cookie.setMaxAge(7 * 24 * 60 * 60); // expires in 7 days
cookie.setSecure(true);
cookie.setHttpOnly(true);

cookie.setPath("/"); // global cookie accessible every where

//add cookie to response
response.addCookie(cookie);
```



## 删除Cookie

```java
// create a cookie
Cookie cookie = new Cookie("username", null);
cookie.setMaxAge(0);
cookie.setSecure(true);
cookie.setHttpOnly(true);
cookie.setPath("/");

//add cookie to response
response.addCookie(cookie);
```

