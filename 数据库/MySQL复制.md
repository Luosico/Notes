# MySQL复制



## 1. 主从搭建

**搭建前需确保主从数据库都有相应的database和table，从数据库不会自动创建database和table，只会更新表中的数据**

**搭建前，master中存在的table，slave不会自动创建table**

**搭建完成后，master新建table，slave会也会自动创建table**

### 1. 配置 master

编辑 **/etc/mysql/my.cnf**, 在`[mysqld]` 中添加以下内容：

```bash
# 定义二进制文件名称，缺省默认为主机名
log-bin=master-bin
# 服务器id，必须保证唯一
server-id=1
# 指定复制的database，缺省为所有database
binlog-do-db=db_test
# 复制方式 row statement mixed
#binlog-format=statement
```



**授权**

```bash
# 创建用户
create user 'slave'@'%' identified by '123456';
# 授权
grant replication slave,replication client on *.* to 'root'@'%';
# 刷新权限
flush privileges;
# 查看用户权限
show grants for 'root'@'%';

#  Authentication requires secure connection 报错解决
alter user 'root'@'%' identified with mysql_native_password by '123456';
```



**重启mysql，查看状态**

```
show master status;
```

![image-20210530220255063](https://raw.githubusercontent.com/Luosico/Typora/main/img/20210530220347.png)



### 2. 配置 slave

编辑 **/etc/mysql/my.cnf**, 在`[mysqld]` 中添加以下内容：

```bash
log-bin=master-bin
server-id=2
# only read
read_only=1
```



**设置 master**

```bash
change master to master_host='mysql1',master_user='root',master_password='123456',master_port=3306,master_log_file='master-bin.000002',master_log_pos=0;
```



**重启mysql，开始复制**

```bash
start slave;

# 查看状态和信息，若有误这里会显示报错信息
show slave status\G
```



### master 已经有数据

**进行数据备份同步**

```bash
1. 登录master，执行锁表操作
mysql -uroot -p
FLUSH TABLES WITH READ LOCK;
2. 将master中需要同步的db的数据dump出来
mysqldump -uroot -p school > school.dump
3. 将数据导入slave
mysql -uroot -h172.17.0.4 -p school < school.dump
4. 解锁master
UNLOCK TABLES;
```

