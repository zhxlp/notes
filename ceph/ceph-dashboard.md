# Ceph Dashboard

## 启用 Dashboard

- 安装软件包

  ```bash
  yum install -y ceph-mgr-dashboard
  ```

- 启用 dashboard module

  ```bash
  ceph mgr module enable dashboard
  ```

- SSL

  ```bash
  # 自动生成SSL证书并配置
  ceph dashboard create-self-signed-cert
  # 手动配置指定SSL证书
  ceph dashboard set-ssl-certificate -i dashboard.crt
  ceph dashboard set-ssl-certificate-key -i dashboard.key
  # 禁用SSL
  ceph config set mgr mgr/dashboard/ssl false
  ```

- 放行端口

  ```bash
  # 如果配置了SSL,放行8433端口
  firewall-cmd --add-port=8433/tcp
  firewall-cmd --add-port=8433/tcp --permanent
  # 如果禁用了SSL,放行8080端口
  firewall-cmd --add-port=8080/tcp
  firewall-cmd --add-port=8080/tcp --permanent
  ```

- 创建管理员用户

  ```bash
  # ceph dashboard ac-user-create <username> <password> administrator
  ceph dashboard ac-user-create admin admin administrator
  ```

## 启用 Grafana 嵌入仪表板

### 安装 Grafana

- 安装软件包

  ```bash
  wget https://dl.grafana.com/oss/release/grafana-6.7.1-1.x86_64.rpm
  yum install grafana-6.7.1-1.x86_64.rpm
  ```

- 启用 ceph grafana 模板

  编辑`/etc/grafana/provisioning/dashboards/sample.yaml`文件

  ```properties
  apiVersion: 1

  providers:
   - name: 'default'
     orgId: 1
     folder: ''
     folderUid: ''
     type: file
     options:
       path: /etc/grafana/dashboards
  ```

- 允许 Grafana 被嵌套

  修改`/etc/grafana/grafana.ini`文件中`allow_embedding`值为`true`

- 启用匿名模式

  修改`/etc/grafana/grafana.ini`文件中的如下内容

  ```properties
  [auth.anonymous]
  enabled = true
  org_name = Main Org.
  org_role = Viewer
  ```

- 启动并设置开机自启

  ```bash
  systemctl restart grafana-server.service
  systemctl enable grafana-server.service
  ```

- 放行端口

  ```bash
  firewall-cmd --add-port=3000/tcp
  firewall-cmd --add-port=3000/tcp --permanent
  ```

- 安装 vonage-status-panel 和 grafana-piechart-panel 插件

  ```bash
  grafana-cli plugins install vonage-status-panel
  grafana-cli plugins install grafana-piechart-panel
  systemctl restart grafana-server.service
  ```

### 安装 node_exporter

- 安装可执行文件

  ```bash
  wget https://github.com/prometheus/node_exporter/releases/download/v0.18.1/node_exporter-0.18.1.linux-amd64.tar.gz
  tar xvf node_exporter-0.18.1.linux-amd64.tar.gz
  mv node_exporter-0.18.1.linux-amd64/node_exporter /usr/bin/
  ```

- 配置 systemd 服务

  创建`/usr/lib/systemd/system/node_exporter.service`文件

  ```properties
  [Unit]
  Description=Node Exporter

  [Service]
  #User=node_exporter
  ExecStart=/usr/bin/node_exporter
  Restart=always

  [Install]
  WantedBy=multi-user.target
  ```

- 启动服务并设置开机自启

  ```bash
  systemctl daemon-reload
  systemctl restart node_exporter.service
  systemctl enable node_exporter.service
  ```

- 放行端口

  ```bash
  firewall-cmd --add-port=9100/tcp
  firewall-cmd --add-port=9100/tcp --permanent
  ```

### 安装 prometheus

- 安装可执行文件

  ```bash
  # 下载文件
  wget https://github.com/prometheus/prometheus/releases/download/v2.17.1/prometheus-2.17.1.linux-amd64.tar.gz
  # 解压文件
  tar xvf prometheus-2.17.1.linux-amd64.tar.gz
  # 移动可执行文件到运行目录
  mv prometheus-2.17.1.linux-amd64/prometheus /usr/bin/
  mv prometheus-2.17.1.linux-amd64/promtool /usr/bin/
  mv prometheus-2.17.1.linux-amd64/tsdb /usr/bin/
  # 配置文件
  mkdir -p /etc/prometheus/
  mv prometheus-2.17.1.linux-amd64/prometheus.yml /etc/prometheus/
  mkdir -p /etc/default
  tee /etc/default/prometheus << EOF
  PROMETHEUS_OPTS='--config.file=/etc/prometheus/prometheus.yml --storage.tsdb.path=/var/lib/prometheus/data'
  EOF
  # 其它文件
  mkdir -p /usr/share/prometheus/
  mv prometheus-2.17.1.linux-amd64/consoles /usr/share/prometheus/
  mv prometheus-2.17.1.linux-amd64/console_libraries /usr/share/prometheus/
  mkdir -p /var/lib/prometheus
  ```

- 配置 systemd 服务

  创建`/usr/lib/systemd/system/prometheus.service`文件

  ```properties
  [Unit]
  Description=The Prometheus 2 monitoring system and time series database.
  Documentation=https://prometheus.io
  After=network.target

  [Service]
  EnvironmentFile=-/etc/default/prometheus
  # User=prometheus
  ExecStart=/usr/bin/prometheus \
            --web.console.libraries=/usr/share/prometheus/console_libraries \
            --web.console.templates=/usr/share/prometheus/consoles \
            $PROMETHEUS_OPTS
  ExecReload=/bin/kill -HUP $MAINPID
  Restart=always
  LimitNOFILE=65536

  [Install]
  WantedBy=multi-user.target
  ```

- 启动服务并设置开机自启

  ```bash
  systemctl daemon-reload
  systemctl restart prometheus.service
  systemctl enable prometheus.service
  ```

- 放行端口

  ```bash
  firewall-cmd --add-port=9090/tcp
  firewall-cmd --add-port=9090/tcp --permanent
  ```

### 配置 ceph

- 启动 Ceph Exporter

  ```bash
  ceph mgr module enable prometheus
  ```

- 设置 grafana-api-url

  ```bash
  ceph dashboard set-grafana-api-url 'http://192.168.200.11:3000'
  ```

- 关闭 ssl 校验

  ```bash
  ceph dashboard set-grafana-api-ssl-verify False
  ```

- 配置 prometheus

  编辑`/etc/prometheus/prometheus.yml`

  ```properties
  global:
    scrape_interval:     15s
    evaluation_interval: 15s

  scrape_configs:
    - job_name: 'node'
      file_sd_configs:
        - files:
          - node_targets.yml
    - job_name: 'ceph'
      honor_labels: true
      file_sd_configs:
        - files:
          - ceph_targets.yml
  ```

  编辑`/etc/prometheus/node_targets.yml`

  ```properties
  [
      {
          "targets": [ "node1:9100" ],
          "labels": {
              "instance": "node1"
          }
      },
      {
          "targets": [ "node2:9100" ],
          "labels": {
              "instance": "node2"
          }
      },
      {
          "targets": [ "node3:9100" ],
          "labels": {
              "instance": "node3"
          }
      }
  ]

  ```

  编辑`/etc/prometheus/ceph_targets.yml`

  ```properties
  [
      {
          "targets": [ "node1:9283" ],
          "labels": {}
      }
  ]

  ```

  重启 prometheus

  ```bash
  systemctl restart prometheus.service
  ```

- 配置 Grafana

  登录 Grafana, 创建**prometheus** data sourc
