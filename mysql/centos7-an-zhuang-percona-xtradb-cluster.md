# CentOS7 安装 Percona XtraDB Cluster

- 关闭 SeLinux

  [all]

  ```bash
  setenforce 0
  sed -i 's/^SELINUX *=.*/SELINUX=disabled/g' /etc/selinux/config
  ```

- 端口放行

  [all]

  ```bash
  firewall-cmd --add-port=3306/tcp --add-port=4444/tcp --add-port=4567/tcp --add-port=4568/tcp
  firewall-cmd --add-port=3306/tcp --add-port=4444/tcp --add-port=4567/tcp --add-port=4568/tcp --permanent
  ```

- 安装`percona`库

  [all]

  ```bash
  yum install https://repo.percona.com/yum/percona-release-latest.noarch.rpm
  ```

- 安装软件包

  [all]

  ```bash
  yum install Percona-XtraDB-Cluster-57
  ```

- 配置集群

  [all]

  ```bash
  # 指定Galera库的路径
  # Debian或Ubuntu： /usr/lib/libgalera_smm.so
  # 红帽或CentOS： /usr/lib64/galera3/libgalera_smm.so
  crudini --set /etc/percona-xtradb-cluster.conf.d/wsrep.cnf 'mysqld' 'wsrep_provider' '/usr/lib64/galera3/libgalera_smm.so'

  # 默认情况下，Percona XtraDB群集使用Percona XtraBackup进行状态快照传输（SST）。wsrep_sst_method=xtrabackup-v2强烈建议进行设置。此方法需要在初始节点上设置用户SST。使用wsrep_sst_auth变量提供SST用户凭据。
  crudini --set /etc/percona-xtradb-cluster.conf.d/wsrep.cnf 'mysqld' 'wsrep_sst_method' 'xtrabackup-v2'

  # 指定认证凭证SST 作为<sst_user>:<sst_pass>。您必须在引导第一个节点时创建此用户 并为其提供必要的权限
  crudini --set /etc/percona-xtradb-cluster.conf.d/wsrep.cnf 'mysqld' 'wsrep_sst_auth' 'sstuser:passw0rd'

  # PXC严格模式默认启用并设置为ENFORCING，这会阻止在Percona XtraDB Cluster中使用实验和不支持的功能
  crudini --set /etc/percona-xtradb-cluster.conf.d/wsrep.cnf 'mysqld' 'pxc_strict_mode' 'ENFORCING'

  # Galera仅支持行级复制，因此请设置binlog_format=ROW
  crudini --set /etc/percona-xtradb-cluster.conf.d/wsrep.cnf 'mysqld' 'binlog_format' 'ROW'

  # Galera完全支持InnoDB存储引擎。它无法与MyISAM或任何其他非事务性存储引擎一起正常工作。将此变量设置为default_storage_engine=InnoDB。
  crudini --set /etc/percona-xtradb-cluster.conf.d/wsrep.cnf 'mysqld' 'default_storage_engine' 'InnoDB'

  # Galera仅支持2InnoDB的interleaved（）锁定模式。设置传统（0）或连续（1）锁定模式可能会导致复制因未解决的死锁而失败。将此变量设置为innodb_autoinc_lock_mode=2
  crudini --set /etc/percona-xtradb-cluster.conf.d/wsrep.cnf 'mysqld' 'innodb_autoinc_lock_mode' '2'

  # 指定群集的逻辑名称。对于群集中的所有节点，它必须相同
  crudini --set /etc/percona-xtradb-cluster.conf.d/wsrep.cnf 'mysqld' 'wsrep_cluster_name' 'pxc-cluster'

  # 指定群集中节点的IP地址。节点加入群集至少需要一个，但建议列出所有节点的地址。这样，如果列表中的第一个节点不可用，则加入节点可以使用其他地址。
  crudini --set /etc/percona-xtradb-cluster.conf.d/wsrep.cnf 'mysqld' 'wsrep_cluster_address' 'gcomm://10.0.100.61,10.0.100.62,10.0.100.63'
  ```

  [pcx1]

  ```bash
  # 指定每个节点的逻辑名称。如果未指定此变量，则将使用主机名
  crudini --set /etc/percona-xtradb-cluster.conf.d/wsrep.cnf 'mysqld' 'wsrep_node_name' 'pxc1'

  # 指定此特定节点的IP地址
  crudini --set /etc/percona-xtradb-cluster.conf.d/wsrep.cnf 'mysqld' 'wsrep_node_address' '10.0.100.61'
  ```

  [pcx2]

  ```bash
  # 指定每个节点的逻辑名称。如果未指定此变量，则将使用主机名
  crudini --set /etc/percona-xtradb-cluster.conf.d/wsrep.cnf 'mysqld' 'wsrep_node_name' 'pxc2'

  # 指定此特定节点的IP地址
  crudini --set /etc/percona-xtradb-cluster.conf.d/wsrep.cnf 'mysqld' 'wsrep_node_address' '10.0.100.62'
  ```

  [pcx3]

  ```bash
  # 指定每个节点的逻辑名称。如果未指定此变量，则将使用主机名
  crudini --set /etc/percona-xtradb-cluster.conf.d/wsrep.cnf 'mysqld' 'wsrep_node_name' 'pxc3'

  # 指定此特定节点的IP地址
  crudini --set /etc/percona-xtradb-cluster.conf.d/wsrep.cnf 'mysqld' 'wsrep_node_address' '10.0.100.63'
  ```

- 引导集群启动

  [pcx1]

  ```bash
  systemctl restart mysql@bootstrap.service
  systemctl status mysql@bootstrap.service
  ```

- 查看初始化密码

  [pcx1]

  ```bash
  sudo grep 'temporary password' /var/log/mysqld.log
  ```

- 安全设置

  [pcx1]

  ```bash
  mysql_secure_installation
  ```

- 添加 SST 用户

  [pcx1]

  > 在配置文件的`wsrep_sst_auth`项中配置的用户名和密码

  ```bash
  mysql -uroot -p

  CREATE USER 'sstuser'@'localhost' IDENTIFIED BY 'passw0rd';
  GRANT RELOAD, LOCK TABLES, PROCESS, REPLICATION CLIENT ON *.* TO 'sstuser'@'localhost';
  FLUSH PRIVILEGES;
  ```

- 依次启动其它节点

  [other]

  ```bash
  systemctl restart mysqld.service
  systemctl status mysqld.service
  ```

  > 注意：连接数据库,查询当前节点状态`wsrep_local_state_comment`,结果为`Synced`才能在下一个节点启动数据库。

  - 重启第一节点

    [pcx1]

    ```bash
    systemctl stop mysql@bootstrap.service
    systemctl restart mysqld.service
    systemctl status mysqld.service
    ```
