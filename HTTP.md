# 										HTTP知识
## 1. Cookie与Session
1. cookie存在于客户端，Session存在于Tomcat(服务器)内存中  
2. 初次访问服务器（Tomcat），服务器会为每个用户生成 JSESSIONID,用来标识每个用户，并通知客户端设置这个值的cookie。当用户再次访问时发送的cookie带上JSESSIONID就不需要用户手动验证而是服务器自动验证直接进入网页（即将相关信息存在session中，通过JSESSIONID的标识从服务器内存中取出需要的数据，从而完成验证操作）。

## 2. 下载文件
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

## 3.URL的提交

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

  