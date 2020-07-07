# 										HTTP知识
## 1. Cookie与Session
1. cookie存在于客户端，Session存在于Tomcat(服务器)内存中 ，以（key, value）的形式存储(sessionId,session)
2. 初次访问服务器（Tomcat），服务器会为每个用户生成 JSESSIONID,用来标识每个用户，并通知客户端设置这个值的cookie。当用户再次访问时发送的cookie带上JSESSIONID就不需要用户手动验证而是服务器自动验证直接进入网页（即将相关信息存在session中，通过JSESSIONID的标识从服务器内存中取出需要的数据，从而完成验证操作）。



### 2、HTTP各版本的差别（1.0  1.1  2.0）

- **HTTP1.0和HTTP1.1的一些区别**
  1. **缓存处理**，在HTTP1.0中主要使用header里的If-Modified-Since,Expires来做为缓存判断的标准，HTTP1.1则引入了更多的缓存控制策略例如Entity tag，If-Unmodified-Since, If-Match, If-None-Match等更多可供选择的缓存头来控制缓存策略。
  2. **带宽优化及网络连接的使用**，HTTP1.0中，存在一些浪费带宽的现象，例如客户端只是需要某个对象的一部分，而服务器却将整个对象送过来了，并且不支持断点续传功能，HTTP1.1则在请求头引入了range头域，它允许只请求资源的某个部分，即返回码是206（Partial Content），这样就方便了开发者自由的选择以便于充分利用带宽和连接。
  3. **错误通知的管理**，在HTTP1.1中新增了24个错误状态响应码，如409（Conflict）表示请求的资源与资源的当前状态发生冲突；410（Gone）表示服务器上的某个资源被永久性的删除。
  4. **Host头处理**，在HTTP1.0中认为每台服务器都绑定一个唯一的IP地址，因此，请求消息中的URL并没有传递主机名（hostname）。但随着虚拟主机技术的发展，在一台物理服务器上可以存在多个虚拟主机（Multi-homed Web Servers），并且它们共享一个IP地址。HTTP1.1的请求消息和响应消息都应支持Host头域，且请求消息中如果没有Host头域会报告一个错误（400 Bad Request）。
  5. **长连接**，HTTP 1.1支持长连接（PersistentConnection）和请求的流水线（Pipelining）处理，在一个TCP连接上可以传送多个HTTP请求和响应，减少了建立和关闭连接的消耗和延迟，在HTTP1.1中默认开启Connection： keep-alive，一定程度上弥补了HTTP1.0每次请求都要创建连接的缺点。
- **HTTP2.0和HTTP1.X相比的新特性**
  - **新的二进制格式**（Binary Format），HTTP1.x的解析是基于文本。基于文本协议的格式解析存在天然缺陷，文本的表现形式有多样性，要做到健壮性考虑的场景必然很多，二进制则不同，只认0和1的组合。基于这种考虑HTTP2.0的协议解析决定采用二进制格式，实现方便且健壮。
  - **多路复用**（MultiPlexing），即连接共享，即每一个request都是是用作连接共享机制的。一个request对应一个id，这样一个连接上可以有多个request，每个连接的request可以随机的混杂在一起，接收方可以根据request的 id将request再归属到各自不同的服务端请求里面。
  - **header压缩**，如上文中所言，对前面提到过HTTP1.x的header带有大量信息，而且每次都要重复发送，HTTP2.0使用encoder来减少需要传输的header大小，通讯双方各自cache一份header fields表，既避免了重复header的传输，又减小了需要传输的大小。
  - **服务端推送**（server push），同SPDY一样，HTTP2.0也具有server push功能。



## 3. 下载文件
 通过 Response 一个输出流即可完成下载文件  

    @RequestMapping()  
    @ResponseBody
    public String export_excel(HttpServletResponse httpServletResponse) throws Exception {
        httpServletResponse.setContentType("application/binary;charset=UTF-8");
        httpServletResponse.setHeader("Content-Disposition", "attachment;fileName=" + URLEncoder.encode("TickNet成员表.xls", "UTF-8"));
        ServletOutputStream out = httpServletResponse.getOutputStream();
    
        String[] titles = {"帐号", "姓名", "性别", "年级专业", "QQ", "电话", "项目兴趣", "部门", "申请岗位", "曾获荣誉", "注册时间"};
        userService.export_excel(titles,out);
        //System.out.println("success");
        out.flush();
        out.close();
        return "success";
    
    }



## 4.URL的提交

- 相对路径：继承了父文档的部分信息

  ​	一般提交的URL会使用相对路径，只改变路径的最后一个`/`后的值

```
若访问的是http://www.baidu.com/test/hello
<a href="world">link</a> //其访问的真正路径是http://www.baidu.com/test/world
```

- 绝对路径（这里不是真正意义上的）

  ​	链接以`/`开头，那么它是相对与文档的根目录，而不是相对与当前文件

  ```
  原路径同上
  <a href="/world">link</a> //其真正访问路径是http://www.baidu.com/world
  ```

  