# Ubuntu配置jdk



## 1. 下载JDK

前往 http://jdk.java.net/ 下载openjdk



```bash
# 解压
tar -zxvf jdk-11.0.7_linux-x64_bin.tar.gz

# 移动位置
cp -r ./jdk-11.0.7 /usr/java/
```



## 2. 设置环境变量

**编辑 `etc/profile`，增加**

```bash
export JAVA_HOME=/usr/java/jdk-11.0.7
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export PATH=$PATH:$JAVA_HOME/bin
```

这样设置每次都需要重新 `source /etc/profile`



**在每个用户的 .bashrc 上增加**

```bash
# 每次bash登录都会执行
source /etc/profile
```

