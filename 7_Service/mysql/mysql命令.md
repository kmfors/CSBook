## binlog 文件

当前二进制日志列表

```mysql
mysql> SHOW BINARY LOGS;
+---------------+-----------+-----------+
| Log_name      | File_size | Encrypted |
+---------------+-----------+-----------+
| binlog.000001 |       505 | No        |
| binlog.000002 |       595 | No        |
+---------------+-----------+-----------+
2 rows in set (0.00 sec)
```

如果系统没有启用二进制日志记录，则会看到以下错误消息。

```mysql
You are not using binary logging
```

mysql binlog 位置：默认情况下，二进制日志文件位于 `/var/lib/mysql` 目录下

配置的二进制日志的相关信息：

```mysql
mysql> show variables like '%log_bin%';
+---------------------------------+-----------------------------+
| Variable_name                   | Value                       |
+---------------------------------+-----------------------------+
| log_bin                         | ON                          |
| log_bin_basename                | /var/lib/mysql/binlog       |
| log_bin_index                   | /var/lib/mysql/binlog.index |
| log_bin_trust_function_creators | OFF                         |
| sql_log_bin                     | ON                          |
+---------------------------------+-----------------------------+
5 rows in set (0.01 sec)
```


MySQL 来定期清理 binlog 日志：在 MySQL 配置文件（my.cnf 或 my.ini）中添加以下行, 之后重启 MySQL 服务使更改生效。这样，MySQL 将会自动删除超过 7 天的 binlog 日志文件。

```text
[mysqld]
expire_logs_days = 7
```

重启 mysql 服务：

```shell
sudo service mysql restart
# sudo systemctl restart mysql
```

https://blog.csdn.net/weixin_45112997/article/details/128287422  



## mysql 忘记密码

如果您忘记了 MySQL 的 root 密码，可以按照以下步骤重置密码：

1. 停止 MySQL 服务：在 Linux 上，可以使用 `sudo systemctl stop mysql` 或 `sudo /etc/init.d/mysql stop`。

2. 启动 MySQL 的安全模式（跳过权限表）：在 Linux 上，可以使用 `sudo mysqld_safe --skip-grant-tables &`。

3. 登录到 MySQL：运行 `mysql -u root`。

4. 在 MySQL 命令行中，执行以下 SQL 命令来设置新密码：

   ```mysql
   FLUSH PRIVILEGES;
   ALTER USER 'root'@'localhost' IDENTIFIED BY '新密码';
   ```

5. 退出 MySQL 命令行：`quit`
6. 停止安全模式下的 MySQL 服务（可以在第 2 步时直接使用 CTRL+C 来停止）。
7. 重新启动 MySQL 服务：在 Linux 上，可以使用 `sudo systemctl start mysql` 或 `sudo /etc/init.d/mysql start`。

现在您应该能够使用新设置的密码登录到 MySQL 了。如果您使用的是不同的操作系统或 MySQL 的安装方式不同，步骤可能会有所变化。

