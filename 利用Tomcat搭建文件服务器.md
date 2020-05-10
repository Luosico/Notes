# <center>利用Tomcat搭建文件服务器</center>

### 修改Tomcat web.xml,允许Tomcat列举文件
		<init-param>
            <param-name>listings</param-name>
            <param-value>true</param-value> //默认为false
        </init-param>

### 设置Tomcat虚拟目录，在conf/Catalina/localhost下新建一个xml文件，内容如下：
		<?xml version="1.0" encoding="UTF-8"?>
		<Context path="/files" reloadable="true" docBase="D:\Java" crossContext="true"/>
		
path指的是URL后的路径，如：`localhost:80/files`  ，path后的名称必须和xml的名称一致
docBase指的是文件的访问路径

### 还需修改/bin/catalina.sh，不然上传的文件没权限访问
		if [ -z "$UMASK" ]; then
    		UMASK="0022" //初始为0027
		fi
		umask $UMASK


![Y3Ts3j.png](https://s1.ax1x.com/2020/05/10/Y3Ts3j.png)
