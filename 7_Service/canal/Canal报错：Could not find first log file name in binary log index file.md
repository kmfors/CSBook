错误描述：canal 服务长时间暂停后，再次启动无法找到 mysql binlog 的问题

# 一、错误信息

```
at com.alibaba.otter.canal.parse.inbound.mysql.dbsync.DirectLogFetcher.fetch(DirectLogFetcher.java:102) ~[canal.parse-1.1.0.jar:na]
        at com.alibaba.otter.canal.parse.inbound.mysql.MysqlConnection.dump(MysqlConnection.java:213) ~[canal.parse-1.1.0.jar:na]
        at com.alibaba.otter.canal.parse.inbound.AbstractEventParser$3.run(AbstractEventParser.java:248) ~[canal.parse-1.1.0.jar:na]
        at java.lang.Thread.run(Thread.java:748) [na:1.8.0_181]
2018-12-05 19:50:00.548 [destination = example , address = /192.168.56.104:3306 , EventParser] ERROR com.alibaba.otter.canal.common.alarm.LogAlarmHandler - destination:example[java.io.IOException: Received error packet: errno = 1236, sqlstate = HY000 errmsg = Could not find first log file name in binary log index file
        at com.alibaba.otter.canal.parse.inbound.mysql.dbsync.DirectLogFetcher.fetch(DirectLogFetcher.java:102)
        at com.alibaba.otter.canal.parse.inbound.mysql.MysqlConnection.dump(MysqlConnection.java:213)
        at com.alibaba.otter.canal.parse.inbound.AbstractEventParser$3.run(AbstractEventParser.java:248)
        at java.lang.Thread.run(Thread.java:748)
```

# 二、解决办法

1、关闭 canal 服务

```shell
sh canal/bin/stop.sh
```

2、删除数据消费记录文件

删除上次 canal 服务停止时产生的 meta.bat（数据消费位置文件）

```shell
rm canal/conf/example/meta.bat
```

3、启动 canal 服务

```shell
sh canal/bin/start.sh
```

备注：如果启动服务后还是无法正常读取 mysql binlog，请继续第三步

# 三、修改配置文件文件

```shell
vim canal/conf/example/instance.properties
```

1、找到 mysql binlog 配置

```
# position info
# canal.instance.master.address=127.0.0.1:3306
# canal.instance.master.journal.name=
# canal.instance.master.position=
# canal.instance.master.timestamp=
# canal.instance.master.gtid=
```

2、修改 mysql binlog 位置

```mysql
show master status;
```

![[20240823134439.png]]

```
canal.instance.master.address=xk-test-rds-standard.mysql.rds.aliyuncs.com:3306
canal.instance.master.journal.name=mysql-bin.000007
canal.instance.master.position=157
canal.instance.master.timestamp=
canal.instance.master.gtid=
```

修改字段：canal.instance.master.journal.name、canal.instance.master.position

# 四、重启服务

1、重启服务

```shell
sh canal/bin/startup.sh
```

2、查看服务运行情况

```shell
cd canal/logs/example
tail -f example.log
```

3、查看数据消费情况

增删改查监控的数据表，查看数据监控情况

```shell
cd canal/logs/example
tail -f meta.log
```

