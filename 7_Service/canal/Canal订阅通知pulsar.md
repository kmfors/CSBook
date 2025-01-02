# 一、简介

canal，主要用途是基于 MySQL 数据库增量日志解析，提供增量数据订阅和消费。

基于日志增量订阅和消费的业务包括：

- 数据库镜像
- 数据库实时备份
- 索引构建和实时维护(拆分异构索引、倒排索引等)
- 业务 cache 刷新
- 带业务逻辑的增量数据处理

当前的 canal 支持源端 MySQL 版本包括 5.1.x , 5.5.x , 5.6.x , 5.7.x , 8.0.x
![[20240814211126.png]]



# 二、工作原理

**MySQL 主备复制原理：**

- MySQL master 将数据变更写入二进制日志( binary log, 其中记录叫做二进制日志事件 binary log events，可以通过 show binlog events 进行查看)
- MySQL slave 将 master 的 binary log events 拷贝到它的中继日志(relay log)
- MySQL slave 重放 relay log 中事件，将数据变更反映它自己的数据

**canal 工作原理：**

- canal 模拟 MySQL slave 的交互协议，伪装自己为 MySQL slave ，向 MySQL master 发送 dump 协议
- MySQL master 收到 dump 请求，开始推送 binary log 给 slave (即 canal )
- canal 解析 binary log 对象(原始为 byte 流)



# 三、部署



## 1. 准备

对于自建 MySQL ,需要先开启 Binlog 的写入功能，配置 binlog-format 为 ROW 模式。my.cnf 中配置如下：

```
[mysqld]
log-bin=mysql-bin  # 开启 binlog
binlog-format=ROW  # 选择 ROW 模式
server_id=1001     # 配置 MySQL replaction 需要定义，不要和 canal 的 slaveId 重复
```

创建 MySQL 连接用户 canal 并设置密码，授权该用户具有作为 MySQL slave 的权限；如果账户已存在，可直接授权。

```mysql
CREATE USER canal IDENTIFIED BY 'canal';  
GRANT SELECT, REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'canal'@'%';
-- GRANT ALL PRIVILEGES ON *.* TO 'canal'@'%' ;
FLUSH PRIVILEGES;
```

> [!NOTE]
>
> - my.cnf 文件一般在 `/etc/mysql` 目录下
> - 关闭 binlog：在 my.cnf 下添加 `skip-log-bin`
> - 自删超7天的 binlog：在 my.cnf 下添加 `expire_logs_days=7`



## 2. 下载解压：

下载 canal, 访问 release 页面 , 选择需要的包下载, 如以 1.0.17 版本为例

```shell
wget https://github.com/alibaba/canal/releases/download/canal-1.0.17/canal.deployer-1.0.17.tar.gz

# 解压缩
mkdir /tmp/canal
tar zxvf canal.deployer-$version.tar.gz  -C /tmp/canal
```



## 3. 配置

Canal-**server** 配置修改

```shell
vi conf/canal.properties
```

```shell
# 1、修改为canal服务ip地址
canal.ip = 192.168.150.138

# 2、更改服务模式为pulsarMQ
canal.serverMode = pulsarMQ

# 3、可以设置自定义目录（即instance名称），需要在该目录下添加配置文件instance.properties
canal.destinations = example

# 4、修改为对应的pulsar地址
pulsarmq.serverUrl = pulsar://192.168.150.139:6650
pulsarmq.roleToken =
pulsarmq.topicTenantPrefix = public/default
```

---
Canal-**instance** 配置修改：

```shell
vi conf/example/instance.properties
```

```shell
# 1、改为自己的数据库信息
canal.instance.master.address=192.168.150.140:3306

# 2、改成自己的数据库信息
canal.instance.dbUsername=canal
canal.instance.dbPassword=Canal@123

# 3、需要同步的数据库或数据库表
canal.instance.filter.regex=canal\\..*
# table black regex
canal.instance.filter.black.regex=mysql\\.slave_.*

# 4、自定义pulsar中分区与主题
canal.mq.topic=sync-mysql-bin
canal.mq.partition=0
```



## 4. 启动暂停

```shell
sh bin/startup.sh

sh bin/stop.sh
```



## 5. 日志

server 日志

```
tail logs/canal/canal.log

2013-02-05 22:45:27.967 [main] INFO  com.alibaba.otter.canal.deployer.CanalLauncher - ## start the canal server.
2013-02-05 22:45:28.113 [main] INFO  com.alibaba.otter.canal.deployer.CanalController - ## start the canal server[10.1.29.120:11111]
2013-02-05 22:45:28.210 [main] INFO  com.alibaba.otter.canal.deployer.CanalLauncher - ## the canal server is running now ......
```

---
instancer 日志

```
vi logs/example/example.log

2013-02-05 22:50:45.636 [main] INFO  c.a.o.c.i.spring.support.PropertyPlaceholderConfigurer - Loading properties file from class path resource [canal.properties]
2013-02-05 22:50:45.641 [main] INFO  c.a.o.c.i.spring.support.PropertyPlaceholderConfigurer - Loading properties file from class path resource [example/instance.properties]
2013-02-05 22:50:45.803 [main] INFO  c.a.otter.canal.instance.spring.CanalInstanceWithSpring - start CannalInstance for 1-example 
2013-02-05 22:50:45.810 [main] INFO  c.a.otter.canal.instance.spring.CanalInstanceWithSpring - start successful....
```



# 四、注意事项

查看 binlog 是否启用命令：

```mysql
mysql> show variables like '%log_bin%';
```

---
解释一下配置

```
# table regex
canal.instance.filter.regex=canal\\..*
# table black regex
canal.instance.filter.black.regex=mysql\\.slave_.*
# table field filter(format: schema1.tableName1:field1/field2,schema2.tableName2:field1/field2)
#canal.instance.filter.field=test1.t_product:id/subject/keywords,test2.t_company:id/name/contact/ch
# table field black filter(format: schema1.tableName1:field1/field2,schema2.tableName2:field1/field2)
#canal.instance.filter.black.field=test1.t_product:subject/product_image,test2.t_company:id/name/contact/ch
```

