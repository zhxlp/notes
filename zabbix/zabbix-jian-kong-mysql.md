# Zabbix 监控 MySQL

- 数据库创建 zabbix 用户

  ```sql
  grant usage on *.* to zabbix@127.0.0.1 identified by 'Wg,Rbv]+5nDN';
  flush privileges;
  ```

- 创建账号信息配置文件

  ```bash
  mkdir /var/lib/zabbix
  echo '
  [client]
  user=zabbix
  host=127.0.0.1
  password=Wg,Rbv]+5nDN
  ' > /var/lib/zabbix/.my.cnf
  chown -R zabbix:zabbix /var/lib/zabbix
  ```

- 主机添加`Template DB MySQL`模板

  【管理】【主机】的【点击主机名】【模板】【选择 Template DB MySQL】【添加】【更新】
