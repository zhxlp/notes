# MySql 主从配置

- 环境

  操作系统：CentOS7.6.1810

  MySql: 5.7.26

- 安装数据库

- 修改配置文件

  [all]

  修改`/etc/my.cnf`配置文件，在`[mysqld]`添加如下内容

  ```properties
  # 服务器ID,同一个集群中,不能重复
  server_id = 1
  # 开启日志并命名
  log-bin = mysql-bin
  ```

- 启动服务

  [all]

  ```bash
  systemctl restart mysqld.service
  systemctl status mysqld.service
  ```

- 数据库安全设置

  [master]

  ```bash
  mysql_secure_installation
  ```

- 检查服务

  - 检查数据库 ID

    ```sql
    show variables like 'server_id';
    -- mysql> show variables like 'server_id';
    -- +---------------+-------+
    -- | Variable_name | Value |
    -- +---------------+-------+
    -- | server_id     | 1     |
    -- +---------------+-------+
    -- 1 row in set (0.00 sec)

    ```

  - 检查数据库日志是否开启

    ```sql
    show variables like '%log_bin%';
    --  mysql> show variables like '%log_bin%';
    --  +---------------------------------+--------------------------------+
    --  | Variable_name                   | Value                          |
    --  +---------------------------------+--------------------------------+
    --  | log_bin                         | ON                             |
    --  | log_bin_basename                | /var/lib/mysql/mysql-bin       |
    --  | log_bin_index                   | /var/lib/mysql/mysql-bin.index |
    --  | log_bin_trust_function_creators | OFF                            |
    --  | log_bin_use_v1_row_events       | OFF                            |
    --  | sql_log_bin                     | ON                             |
    --  +---------------------------------+--------------------------------+
    --  6 rows in set (0.00 sec)
    ```

- 创建同步用户

  [master]

  ```sql
  grant replication slave on *.* to 'repl'@'10.0.100.%' identified by '!QAZ2wsx';
  flush privileges;
  ```

- 导出数据库

  [master]

  ```bash
  mysqldump -uroot -p --master-data --all-databases > db-all.sql
  ```

- 导入数据到数据库

  [slave]

  将主库导出的数据库复制到从库服务器上的`~/db-all.sql`位置,登录数据库,执行导入命令进行导入

  ```sql
  source ~/db-all.sql
  ```

- 查看`MASTER_LOG_FILE`和`MASTER_LOG_POS`

  [slave]

  ```bash
  egrep '^CHANGE MASTER TO' ~/db-all.sql
  # CHANGE MASTER TO MASTER_LOG_FILE='mysql-bin.000002', MASTER_LOG_POS=131276061;
  ```

- 设置主从

  [slave]

  ```sql
  CHANGE MASTER TO MASTER_HOST='10.0.100.71',
  MASTER_PORT=3306,
  MASTER_USER='repl',
  MASTER_PASSWORD='!QAZ2wsx',
  MASTER_LOG_FILE='mysql-bin.000002',
  MASTER_LOG_POS=131276061;
  ```

- 启动同步,并查看状态

  [slave]

  ```sql
  start slave;
  show slave status \G;
  --  ******************** 1. row ***************************
  --          Slave_IO_State: Queueing master event to the relay log
  --             Master_Host: 10.0.100.71
  --             Master_User: repl
  --             Master_Port: 3306
  --           Connect_Retry: 60
  --         Master_Log_File: mysql-bin.000002
  --     Read_Master_Log_Pos: 131276061
  --          Relay_Log_File: mysql2-relay-bin.000002
  --           Relay_Log_Pos: 13865689
  --   Relay_Master_Log_File: mysql-bin.000002
  --        Slave_IO_Running: Yes
  --       Slave_SQL_Running: Yes
  --

  ```

  > `Slave_IO_Running` 和`Slave_IO_Running`值为`Yes`表示设置成功
