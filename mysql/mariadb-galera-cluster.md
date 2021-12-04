# MariaDB Galera Cluster

## 安装

1. 安装`mariadb-galera-server`包，CentOS7.3 默认源没有这个包，可以通过安装`centos-release-openstack-newton.noarch`源使用 openstack 里面的包，或者配置

   ```bash
   yum install -y centos-release-openstack-newton.noarch && \
   yum install -y mariadb mariadb-galera-server mariadb-galera-common galera rsync
   ```

2. 在**node1 节点**启动数据库，并运行`mysql_secure_installation`脚本，对数据进行安全配置,之后再停止数据库。

   ```bash
   systemctl start mariadb.service
   mysql_secure_installation
   systemctl stop mariadb.service
   ```

3. 在**所有节点**修改`/etc/my.cnf.d/galera.cnf`文件，参考下列内容修改：

   ```properties
   [mysqld]
   ......
   wsrep_provider = /usr/lib64/galera/libgalera_smm.so
   wsrep_cluster_address = "gcomm://node1,node2,node3"
   wsrep_node_name = node1
   wsrep_node_address=192.168.49.51
   ```

4. 防火墙放行端口。

   ```bash
   firewall-cmd --permanent --add-port=3306/tcp --add-port=4567/tcp --add-port=4444/tcp
   firewall-cmd --reload
   ```

   > 3306，这个是 MariaDB/MySQL 的服务端口，这个都不开那就不用跑 MariaDB/MySQL 服务了。
   > 4567，Galera 做数据复制的通讯和数据传输端口，需要在防火墙放开 TCP 和 UDP
   > 4568，Galera 做增量数据传输使用的端口（Incremental State Transfer, IST），需要防火墙放开 TCP
   > 4444，Galera 做快照状态传输使用的端口（State Snapshot Transfer, SST），需要防火墙放开 TCP

## 启动群集

在**node1 节点**执行

```bash
galera_new_cluster
#或
/usr/libexec/mysqld --wsrep-new-cluster --user=root &
```

在**其它节点**启动数据库

```bash
systemctl start mariadb.service
```

在**node1 节点**登陆数据库，查看集群信息

```sql
SHOW STATUS LIKE 'wsrep%';
```

## 故障恢复

### 部分故障

在故障节点重新启动 MariaDB 即可。

```bash
systemctl start mariadb.service
```

### 全部故障

分析各节点日志，找出最后故障的节点。

```bash
grep "New cluster view" /var/log/mariadb/mariadb.log |awk  -F: 'END { print $1":"$2":"$3 $6":"$7}'

2018-12-26 17:53:10 140056571144960 [Note] WSREP 328316ba-08c2-11e9-bc67-6bc4fabc0830:17, view# -1
```

> “:**17**, view#”
>
> 那个节点中的数字最大，此节点即为最后故障节点

在**最后故障**节点

```bash
galera_new_cluster
```

在其它节点

```bash
systemctl start mariadb.service
```
