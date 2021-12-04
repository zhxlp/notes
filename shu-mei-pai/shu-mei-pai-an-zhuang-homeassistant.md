# 树莓派安装 HomeAssistant

## 软件版本

- 操作系统：Raspbian GNU/Linux 10 (buster)

## 安装 HomeAssistant

- 更换阿里源

  编辑`cat /etc/apt/sources.list`文件，内容如下

  ```properties
  deb http://mirrors.aliyun.com/raspbian/raspbian/ buster main contrib non-free rpi
  ```

  更新软件包信息

  ```bash
  apt update
  ```

- 安装依赖

  ```bash
  sudo apt-get install python3 python3-venv python3-pip
  ```

- 创建`homeassistant` 的用户

  ```bash
  sudo useradd -rm homeassistant
  ```

- 创建安装文件夹

  ```bash
  cd /srv
  sudo mkdir homeassistant
  sudo chown homeassistant:homeassistant homeassistant
  ```

- 创建虚拟环境

  ```bash
  sudo su -s /bin/bash homeassistant
  cd /srv/homeassistant
  python3 -m venv .
  source bin/activate
  ```

- 配置阿里 pip 源

  ```bash
  mkdir ~/.pip
  cat > ~/.pip/pip.conf << EOF
  [global]
  index-url = https://mirrors.aliyun.com/pypi/simple/
  [install]
  trusted-host=mirrors.aliyun.com
  EOF
  ```

- 安装 Home Assistant

  ```bash
  pip3 install homeassistant
  ```

- 初次启动 Home assistant

  初次启动 Home assistant，后台会下载依赖,需要等待几分钟后使用浏览器访问`http://树莓派的 IP 地址:8123`，进入 Home Asssitant

  ```bash
  hass
  ```

  > 安装过程会下载许多国外资源，可能会失败，注意查看日志

- 设置为服务

  创建`/etc/systemd/system/home-assistant.service`文件，内容如下

  ```properties
  [Unit]
  Description=Home Assistant
  After=network-online.target

  [Service]
  Type=simple
  User=homeassistant
  ExecStart=/srv/homeassistant/bin/hass -c "/home/homeassistant/.homeassistant"

  [Install]
  WantedBy=multi-user.target
  ```

  重新加载服务,启动并设置自启

  ```bash
  systemctl daemon-reload
  systemctl start home-assistant.service
  systemctl enable home-assistant.service
  ```
