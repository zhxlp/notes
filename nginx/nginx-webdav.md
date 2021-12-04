# Nginx-WebDAV

- 安装依赖包

  ```bash
  yum install -y pcre pcre-devel zlib zlib-devel gcc-c++ openssl openssl-devel libxml2 libxml2-devel libxslt-devel
  ```

- 创建 nginx 用户

  ```bash
  useradd -r -M -s /sbin/nologin nginx
  ```

- 创建临时文件目录

  ```bash
  mkdir -p /var/lib/nginx/tmp
  chown -R nginx:nginx /var/lib/nginx
  ```

- 编译安装

  下载文件[nginx-1.14.2.tar.gz](http://nginx.org/download/nginx-1.14.2.tar.gz)和[nginx-dav-ext-module-3.0.0.tar.gz](https://github.com/arut/nginx-dav-ext-module/archive/v3.0.0.tar.gz)。

  ```bash
  # 解压文件
  tar -xvf nginx-1.14.2.tar.gz -C /usr/local/src/
  tar -xvf tar -xvf nginx-dav-ext-module-3.0.0.tar.gz -C /usr/local/src/
  # 编译安装
  cd /usr/local/src/nginx-1.14.2
  ./configure  --prefix=/usr/share/nginx \
  --sbin-path=/usr/sbin/nginx \
  --modules-path=/usr/lib64/nginx/modules \
  --conf-path=/etc/nginx/nginx.conf \
  --error-log-path=/var/log/nginx/error.log \
  --http-log-path=/var/log/nginx/access.log \
  --http-client-body-temp-path=/var/lib/nginx/tmp/client_body \
  --http-proxy-temp-path=/var/lib/nginx/tmp/proxy \
  --http-fastcgi-temp-path=/var/lib/nginx/tmp/fastcgi \
  --http-uwsgi-temp-path=/var/lib/nginx/tmp/uwsgi \
  --http-scgi-temp-path=/var/lib/nginx/tmp/scgi \
  --pid-path=/run/nginx.pid \
  --lock-path=/run/lock/subsys/nginx \
  --user=nginx \
  --group=nginx \
  --with-http_ssl_module \
  --with-http_dav_module \
  --add-module=/usr/local/src/nginx-dav-ext-module-3.0.0
  make && make install
  ```

- 配置 nginx

  ```bash
  server {
    listen 8888;

    location / {
      root /data/www;
      autoindex on;
      autoindex_localtime on;
      charset utf-8,gbk;
      dav_methods PUT DELETE MKCOL COPY MOVE;
      dav_ext_methods PROPFIND OPTIONS;
      create_full_put_path on;
      # 配置最大上传文件
      client_max_body_size 100M;
      # 配置临时文件位置，需要于上传目录在同一个分区
      client_body_temp_path /data/www/temp;
      dav_access user:rw group:r all:r;
      auth_basic "Authorized Users Only";
      auth_basic_user_file /etc/nginx/.htpasswd;
    }
  }
  ```

- 安装 http 工具

  ```bash
  yum -y install httpd-tools
  ```

- 创建用户

  ```bash
  htpasswd -c /etc/nginx/.htpasswd user1
  htpasswd /etc/nginx/.htpasswd user2
  ```

- 修改密码文件权限

  ```bash
  chown nginx:nginx /etc/nginx/.htpasswd
  chmod 600 /etc/nginx/.htpasswd
  ```
