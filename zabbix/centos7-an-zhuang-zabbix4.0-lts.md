# CentOS7 安装 Zabbix4.0 LTS

## Yum 安装

- 安装 Zabbix 仓库

  ```bash
  yum install -y https://repo.zabbix.com/zabbix/4.0/rhel/7/x86_64/zabbix-release-4.0-2.el7.noarch.rpm
  yum clean all
  yum makecache
  ```

- 安装 Zabbix 服务端、web 端和代理

  ```bash
  yum -y install zabbix-server-mysql zabbix-web-mysql zabbix-agent
  ```

- 创建数据库及用户

  ```sql
  create database zabbix character set utf8 collate utf8_bin;
  grant all privileges on zabbix.* to zabbix@localhost identified by 'password';
  flush privileges;
  ```

- 导入数据库

  ```bash
  zcat /usr/share/doc/zabbix-server-mysql*/create.sql.gz | mysql -uzabbix -p zabbix
  ```

- 修改配置文件

  编辑`/etc/zabbix/zabbix_server.conf`文件，修改数据库密码

  ```properties
  DBPassword=password
  ```

  编辑`/etc/httpd/conf.d/zabbix.conf`文件，修改时区

  ```properties
  php_value date.timezone Asia/Shanghai
  ```

- 启动服务，并设置开机自启

  ```bash
  systemctl restart zabbix-server zabbix-agent httpd
  systemctl enable zabbix-server zabbix-agent httpd
  ```

- 防火墙放行

  ```bash
  firewall-cmd --add-port=10051/tcp --add-port=80/tcp
  firewall-cmd --add-port=10051/tcp --add-port=80/tcp --permanent
  ```

- 配置 web 界面

  浏览器访问 http://server_ip_or_name/zabbix/

  用户名：Admin

  密码：zabbix

## 中文乱码

- 拷贝 windows 系统字体微软雅黑`msyh.ttf`文件到`/usr/share/zabbix/assets/fonts`目录。

- 修改`/usr/share/zabbix/include/defines.inc.php`文件，将`graphfont`替换为`msyh`.

  > define('ZBX_GRAPH_FONT_NAME', 'graphfont');
  >
  > define('ZBX_FONT_NAME', 'graphfont');

## Agent 安装

- 安装 Zabbix 仓库

  ```bash
  yum install -y https://repo.zabbix.com/zabbix/4.0/rhel/7/x86_64/zabbix-release-4.0-2.el7.noarch.rpm
  yum clean all
  yum makecache
  ```

- 安装 Zabbix Agent

  ```bash
  yum -y install zabbix-agent
  ```

- 修改配置文件

  编辑`/etc/zabbix/zabbix_agentd.conf`文件

  ```properties
  # Passive checks related
  # 被动模式,服务器访问agent获取信息
  # Server配置可以访问agent的IP或主机名(Server端IP或者主机名称)列表,逗号分隔
  Server=
  # Active checks related
  # 主动模式,agent主动向服务端发送信息
  # ServerActive配置服务端的IP或者主机名列表,逗号分隔
  ServerActive=
  # Hostname配置主动模式时，服务端自动发现显示的主机名称
  Hostname=
  ```

- 启动服务并设置开机自启

```bash
systemctl restart zabbix-agent.service
systemctl enable zabbix-agent.service
```

- 防火墙放行

  ```bash
  firewall-cmd --add-port=10050/tcp
  firewall-cmd --add-port=10050/tcp --permanent
  ```

## 结合 Grafana

- 安装 Grafana

  ```bash
  yum install https://dl.grafana.com/oss/release/grafana-6.3.5-1.x86_64.rpm
  ```

- 安装 Grafana Zabbix 插件

  ```bash
  grafana-cli plugins install alexanderzobnin-zabbix-app
  ```

- 启动 Grafana 并设置开启自启

  ```bash
  systemctl restart grafana-server.service
  systemctl enable grafana-server.service
  ```

- 防火墙放行

  ```bash
  firewall-cmd --add-port=3000/tcp
  firewall-cmd --add-port=3000/tcp --permanent
  ```

- 验证

  浏览器访问 `http://server_ip_or_name:3000`

  用户名：admin

  密码：admin
