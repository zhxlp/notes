# NoVNC 安装

- 环境

  Ubuntu 18.04

- 安装**websockify**

  ```bash
  pip install websockify
  ```

- 下载 novnc 界面

  ```bash
  wget https://github.com/novnc/noVNC/archive/v1.1.0.tar.gz
  ```

- 解压 novnc 到`/usr/share/novnc/`

  ```bash
  tar -xvf noVNC-1.1.0.tar.gz
  mv noVNC-1.1.0 /usr/share/novnc
  ```

- 创建 token

  ```bash
  mkdir -p /etc/novnc/token/
  echo 'test: 172.20.0.245:5900' > /etc/novnc/token/test
  ```

- 注册服务

  ```bash
  echo '[Unit]
  Description=NoVNC Server
  After=syslog.target network.target

  [Service]
  Type=simple
  User=root
  ExecStart=/usr/local/bin/websockify --web /usr/share/novnc/ --target-config=/etc/novnc/token/ --log-file=/var/log/novnc.log 6080
  Restart=on-failure

  [Install]
  WantedBy=multi-user.target
  ' > /etc/systemd/system/novnc.service

  systemctl daemon-reload
  ```

- 设置服务开机自启并启动

  ```bash
  systemctl enable novnc.service
  systemctl restart novnc.service
  ```

- 验证

  浏览器访问`http://IP:6080/vnc.html?path=websockify?token=test`
