# 树莓派安装 NextCloud

- 安装依赖软件

  ```bash
  sudo apt install -y apache2 mariadb-server libapache2-mod-php
  sudo apt install -y php-gd php-json php-mysql php-curl php-mbstring php-intl php-imagick php-xml php-zip
  ```

- 下载源码

  ```bash
  # 进入apache2工作目录
  cd /var/www/html
  # 下载源码
  sudo wget https://download.nextcloud.com/server/releases/nextcloud-15.0.8.zip
  ```

- 解压并设置权限

  ```bash
  sudo unzip nextcloud-15.0.8.zip
  sudo chmod 750 nextcloud -R
  sudo chown www-data:www-data nextcloud -R
  ```

- 创建数据库并设置权限

  ```sql
  create database nextcloud;
  grant all privileges on nextcloud.* to 'nextcloud'@'localhost' identified by 'nextcloud';
  flush privileges;
  ```

- 浏览器访问进行配置

  http://IP/nextcloud
