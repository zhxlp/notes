# CentOS7 安装 Nginx

- 安装依赖包

  ```bash
  yum install -y pcre pcre-devel zlib zlib-devel gcc-c++ openssl openssl-devel libxml2 libxml2-devel libxslt-devel gd-devel perl-devel perl-ExtUtils-Embed GeoIP GeoIP-devel GeoIP-data gperftools redhat-rpm-config wget
  ```

- 创建 `nginx`用户

  ```bash
  useradd -r -d /var/lib/nginx -s /sbin/nologin nginx
  ```

  - 创建配置所需文件夹

  ```bash
  mkdir -p /var/lib/nginx/tmp
  chown -R nginx:nginx /var/lib/nginx
  ```

- 下载源码包，解压并进入解压目录

  ```bash
  wget https://nginx.org/download/nginx-1.12.2.tar.gz
  mv nginx-1.12.2.tar.gz /usr/local/src/
  cd /usr/local/src/
  tar xvf nginx-1.12.2.tar.gz
  cd nginx-1.12.2
  ```

- 配置`nginx`编译信息

  ```bash
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
  --with-file-aio \
  --with-ipv6 \
  --with-http_auth_request_module \
  --with-http_ssl_module \
  --with-http_v2_module \
  --with-http_realip_module \
  --with-http_addition_module \
  --with-http_xslt_module=dynamic \
  --with-http_image_filter_module=dynamic \
  --with-http_geoip_module=dynamic \
  --with-http_sub_module \
  --with-http_dav_module \
  --with-http_flv_module \
  --with-http_mp4_module \
  --with-http_gunzip_module \
  --with-http_gzip_static_module \
  --with-http_random_index_module \
  --with-http_secure_link_module \
  --with-http_degradation_module \
  --with-http_slice_module \
  --with-http_stub_status_module \
  --with-http_perl_module=dynamic \
  --with-mail=dynamic \
  --with-mail_ssl_module \
  --with-pcre \
  --with-pcre-jit \
  --with-stream=dynamic \
  --with-stream_ssl_module \
  --with-google_perftools_module \
  --with-debug \
  --with-cc-opt='-O2 -g -pipe -Wall -Wp,-D_FORTIFY_SOURCE=2 -fexceptions -fstack-protector-strong --param=ssp-buffer-size=4 -grecord-gcc-switches -specs=/usr/lib/rpm/redhat/redhat-hardened-cc1 -m64 -mtune=generic' \
  --with-ld-opt='-Wl,-z,relro -specs=/usr/lib/rpm/redhat/redhat-hardened-ld -Wl,-E'
  ```

- 编译和安装

  ```bash
  make && make install
  ```

- 注册服务

  ```bash
  cat > /usr/lib/systemd/system/nginx.service << EOF
  [Unit]
  Description=The nginx HTTP and reverse proxy server
  After=network.target remote-fs.target nss-lookup.target

  [Service]
  Type=forking
  PIDFile=/run/nginx.pid
  ExecStartPre=/usr/bin/rm -f /run/nginx.pid
  ExecStartPre=/usr/sbin/nginx -t
  ExecStart=/usr/sbin/nginx
  ExecStartPost=/bin/sleep 0.1
  ExecReload=/bin/kill -s HUP $MAINPID
  KillSignal=SIGQUIT
  TimeoutStopSec=5
  KillMode=process
  PrivateTmp=true

  [Install]
  WantedBy=multi-user.target
  EOF
  ```

- 重启加载`systemctl`并启动 nginx。

  ```bash
  systemctl daemon-reload
  systemctl start nginx.service
  ```
