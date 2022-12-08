# CentOS7 安装 odoo13

## 参考资料

https://blog.csdn.net/wuhao0015/article/details/104504769/

## 环境

操作系统：CentOS7.6.1810

odoo:13

## 安装

*   创建 odoo 用户

    ```bash
    adduser --system --shell=/bin/bash --home-dir=/opt/odoo -m odoo
    ```
*   创建配置和日志目录

    ```bash
    mkdir /etc/odoo
    mkdir /var/log/odoo
    ```
*   安装依赖包

    ```bash
    # 安装ius源
    yum install  -y https://centos7.iuscommunity.org/ius-release.rpm
    # 安装依赖
    yum groupinstall -y 'Development Tools'
    yum install -y python36u python36u-devel python36-pillow python36-lxml npm nodejs libxml2-devel libjpeg-devel libxml2 libxslt libxslt-devel wget libpng libjpeg openssl icu libX11 libXext libXrender xorg-x11-fonts-Type1 xorg-x11-fonts-75dpi python3-pip python3-setuptools git openldap-devel
    ```
*   安装 Node 依赖

    ```bash
    npm install -g less less-plugin-clean-css -y
    ```
*   安装**wkhtmltopdf**

    ```bash
    wget https://github.com/wkhtmltopdf/wkhtmltopdf/releases/download/0.12.5/wkhtmltox-0.12.5-1.centos7.x86_64.rpm
    rpm -Uvh wkhtmltox-0.12.5-1.centos7.x86_64.rpm
    ```
*   安装配置**PostgreSQL**

    ```bash
    # 安装 PostgreSQL源
    yum install -y https://yum.postgresql.org/10/redhat/rhel-7-x86_64/pgdg-centos10-10-2.noarch.rpm
    # 安装PostgreSQL服务
    yum install -y postgresql10-server postgresql10
    # 初始化数据库
    /usr/pgsql-10/bin/postgresql-10-setup initdb
    # 启动并设置开机自启
    systemctl restart postgresql-10.service
    systemctl enable postgresql-10.service
    ```
*   创建数据库用户 odoo

    ```bash
    su - postgres -c "createuser -s odoo"
    ```
*   克隆代码

    ```bash
    git clone --depth=1 --branch=13.0 https://github.com/odoo/odoo.git /opt/odoo/odoo
    ```
*   安装项目 python 依赖包

    ```bash
    cd /opt/odoo/odoo
    pip3 install -r requirements.txt
    ```
*   配置目录权限

    ```bash
    chown odoo:odoo /opt/odoo/ -R
    chown odoo:odoo /var/log/odoo/ -R
    ```
*   初始化并创建配置文件

    ```bash
    # 初始化
    su - odoo -c "/opt/odoo/odoo/odoo-bin --addons-path=/opt/odoo/odoo/addons -s --stop-after-init"
    # 移动配置文件
    mv /opt/odoo/.odoorc /etc/odoo/odoo.conf
    # 修改配置文件
    sed -i "s,^\(logfile = \).*,\1"/var/log/odoo/odoo-server.log"," /etc/odoo/odoo.conf
    sed -i "s,^\(proxy_mode = \).*,\1"True"," /etc/odoo/odoo.conf
    ```
*   配置 systemd 服务

    ```bash
    # 链接odoo-bin到可执行目录
    ln -s /opt/odoo/odoo/odoo-bin /usr/bin/odoo
    # 配置systemd
    cp /opt/odoo/odoo/debian/odoo.service /usr/lib/systemd/system/odoo.service
    systemctl daemon-reload
    systemctl restart odoo.service
    systemctl enable odoo.service
    chkconfig --levels 2345 odoo on
    ```

## Nginx 反向代理配置

```properties
upstream odoo {
 server 127.0.0.1:8069;
}

upstream odoo-chat {
 server 127.0.0.1:8072;
}

server {
    server_name odoo.example.com;
    return 301 https://odoo.example.com$request_uri;
}

server {
   listen 443 ssl http2;
   server_name odoo.example.com;

   ssl_certificate /path/to/signed_cert_plus_intermediates;
   ssl_certificate_key /path/to/private_key;
   ssl_session_timeout 1d;
   ssl_session_cache shared:SSL:50m;
   ssl_session_tickets off;

   ssl_dhparam /path/to/dhparam.pem;

   ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
   ssl_ciphers 'ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA:ECDHE-RSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-RSA-AES256-SHA256:DHE-RSA-AES256-SHA:ECDHE-ECDSA-DES-CBC3-SHA:ECDHE-RSA-DES-CBC3-SHA:EDH-RSA-DES-CBC3-SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128-SHA256:AES256-SHA256:AES128-SHA:AES256-SHA:DES-CBC3-SHA:!DSS';
   ssl_prefer_server_ciphers on;

   add_header Strict-Transport-Security max-age=15768000;

   ssl_stapling on;
   ssl_stapling_verify on;
   ssl_trusted_certificate /path/to/root_CA_cert_plus_intermediates;
   resolver 8.8.8.8 8.8.4.4;

   access_log /var/log/nginx/odoo.access.log;
   error_log /var/log/nginx/odoo.error.log;

   proxy_read_timeout 720s;
   proxy_connect_timeout 720s;
   proxy_send_timeout 720s;
   proxy_set_header X-Forwarded-Host $host;
   proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
   proxy_set_header X-Forwarded-Proto $scheme;
   proxy_set_header X-Real-IP $remote_addr;

   location / {
     proxy_redirect off;
     proxy_pass http://odoo;
   }

   location /longpolling {
       proxy_pass http://odoo-chat;
   }

   location ~* /web/static/ {
       proxy_cache_valid 200 90m;
       proxy_buffering    on;
       expires 864000;
       proxy_pass http://odoo;
  }

  # gzip
  gzip_types text/css text/less text/plain text/xml application/xml application/json application/javascript;
  gzip on;
}
```
