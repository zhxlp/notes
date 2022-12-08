# 葫芦儿打包

> 操作系统：CentOS7.6.1810

*   关闭`SELinux`

    ```bash
    setenforce 0
    sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux
    ```
*   创建打包目录

    ```bash
    mkdir -p /data/soft/huluer/{src,python3.6,nginx,redis,uwsgi,celery,backend,frontend}
    ```
*   创建`huluer`用户

    ```bash
    getent group huluer > /dev/null || groupadd -r huluer
    getent passwd huluer > /dev/null || useradd -r -d /data/soft/huluer -g huluer -s /sbin/nologin huluer
    ```
* 上传前端代码到`/data/soft/huluer/frontend`目录
* 上传后端代码到`/data/soft/huluer/backend`目录

## 配置 python3.6

*   安装依赖软件

    ```bash
    yum install -y gcc-c++ valgrind-devel systemtap-sdt-devel \
    bzip2-devel ncurses-devel gdbm-devel sqlite-devel openssl-devel \
    readline-devel zlib-devel xz-devel tk-devel wget
    ```
*   下载 python3.6 源码并解压

    ```bash
    wget https://www.python.org/ftp/python/3.6.8/Python-3.6.8.tgz -P /data/soft/huluer/src/
    cd /data/soft/huluer/src/
    tar xvf Python-3.6.8.tgz
    cd Python-3.6.8
    ```
*   编译并安装

    ```bash
    ./configure --prefix=/data/soft/huluer/python3.6 \
      --enable-ipv6 \
      --with-computed-gotos=yes \
      --with-dbmliborder=gdbm:ndbm:bdb \
      --with-system-expat \
      --with-system-ffi \
      --enable-loadable-sqlite-extensions \
      --with-dtrace \
      --with-valgrind \
      --with-ensurepip \
      --enable-optimizations

      make && make install
    ```
*   验证

    ```bash
    /data/soft/huluer/python3.6/bin/python3.6 -V
    /data/soft/huluer/python3.6/bin/pip3.6 -V
    ```
*   安装葫芦儿 pip 依赖包

    ```bash
    /data/soft/huluer/python3.6/bin/pip3.6 install -r /data/soft/huluer/backend/requirements.txt
    ```

## 配置 nginx

*   安装依赖

    ```bash
    yum install -y gcc-c++ gperftools-devel openssl-devel pcre-devel zlib-devel GeoIP-devel gd-devel perl-devel perl-ExtUtils-Embed libxslt-devel wget
    ```
*   下载源码包并解压

    ```bash
    wget https://nginx.org/download/nginx-1.12.2.tar.gz -P /data/soft/huluer/src/
    cd /data/soft/huluer/src/
    tar xvf nginx-1.12.2.tar.gz
    cd nginx-1.12.2
    ```
*   编译并安装

    ```bash
    ./configure \
    --prefix=/data/soft/huluer/nginx \
    --user=huluer \
    --group=huluer \
    --with-file-aio \
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
    --with-cc-opt='-O2 -g -pipe -Wall -Wp,-D_FORTIFY_SOURCE=2 -fexceptions -fstack-protector-strong --param=ssp-buffer-size=4 -grecord-gcc-switches -m64 -mtune=generic' \
    --with-ld-opt='-Wl,-z,relro -Wl,-E'

    make && make install
    ```
*   创建`huluer-nginx.service`文件

    ```bash
    echo '
    [Unit]
    Description=HuLuEr Nginx
    After=network.target remote-fs.target nss-lookup.target

    [Service]
    Type=forking
    PIDFile=/data/soft/huluer/nginx/logs/nginx.pid
    ExecStartPre=/usr/bin/rm -f /data/soft/huluer/nginx/logs/nginx.pid
    ExecStartPre=/data/soft/huluer/nginx/sbin/nginx -t -c /data/soft/huluer/nginx/conf/nginx.conf
    ExecStart=/data/soft/huluer/nginx/sbin/nginx
    ExecReload=/bin/kill -s HUP $MAINPID
    KillSignal=SIGQUIT
    TimeoutStopSec=5
    KillMode=mixed
    PrivateTmp=true

    [Install]
    WantedBy=multi-user.target

    ' > /data/soft/huluer/nginx/huluer-nginx.service
    ```
*   注册系统服务

    ```bash
    sh -c 'cp -f /data/soft/huluer/nginx/huluer-nginx.service /usr/lib/systemd/system/huluer-nginx.service'
    systemctl daemon-reload
    ```
*   配置`nginx.conf`文件

    ```bash
    echo '
    user huluer;
    worker_processes auto;
    error_log /data/soft/huluer/nginx/logs/error.log;
    pid /data/soft/huluer/nginx/logs/nginx.pid;

    events {
        worker_connections 1024;
    }

    http {
        log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                          '$status $body_bytes_sent "$http_referer" '
                          '"$http_user_agent" "$http_x_forwarded_for"';

        access_log  /data/soft/huluer/nginx/logs/access.log  main;

        sendfile            on;
        tcp_nopush          on;
        tcp_nodelay         on;
        keepalive_timeout   65;
        types_hash_max_size 2048;

        include             /data/soft/huluer/nginx/conf/mime.types;
        default_type        application/octet-stream;

        server {
            listen       80;
            server_name  controller;

            access_log /data/soft/huluer/nginx/logs/huluer-frontend-access.log  main;
        	error_log /data/soft/huluer/nginx/logs/huluer-frontend-error.log;

            location / {
                 alias /data/soft/huluer/frontend/;
            }
        }

        server {
             listen 7777;
             server_name controller;

            access_log /data/soft/huluer/nginx/logs/huluer-backend-access.log  main;
        	error_log /data/soft/huluer/nginx/logs/huluer-backend-error.log;

             location / {
                 include uwsgi_params;
                 uwsgi_pass 127.0.0.1:7776;
             }
        }
    }

    ' > /data/soft/huluer/nginx/conf/nginx.conf
    ```

## 配置 redis

*   下载源码

    ```bash
    wget http://download.redis.io/releases/redis-5.0.5.tar.gz -P /data/soft/huluer/src/
    cd /data/soft/huluer/src/
    tar xvf redis-5.0.5.tar.gz
    cd redis-5.0.5
    ```
*   编译安装

    ```bash
    make MALLOC=jemalloc PREFIX=/data/soft/huluer/redis all
    make MALLOC=jemalloc PREFIX=/data/soft/huluer/redis install
    ```
*   修改配置文件

    ```bash
    sh -c 'cp -f /data/soft/huluer/src/redis-5.0.5/redis.conf /data/soft/huluer/redis/redis.conf'
    sed -i 's/^pidfile .*/pidfile \/data\/soft\/huluer\/redis\/redis\.pid/g' /data/soft/huluer/redis/redis.conf
    sed -i 's/^logfile .*/logfile \/data\/soft\/huluer\/redis\/redis\.log/g' /data/soft/huluer/redis/redis.conf
    sed -i 's/^dir .*/dir \/data\/soft\/huluer\/redis/g' /data/soft/huluer/redis/redis.conf
    ```
*   创建`/data/soft/huluer/redis/bin/redis-shutdown`脚本,内容如下

    ```properties
    #!/bin/bash
    #
    # Wrapper to close properly redis and sentinel
    test x"$REDIS_DEBUG" != x && set -x

    REDIS_CLI=/data/soft/huluer/redis/bin/redis-cli

    # Retrieve service name
    SERVICE_NAME="$1"
    if [ -z "$SERVICE_NAME" ]; then
       SERVICE_NAME=redis
    fi

    # Get the proper config file based on service name
    CONFIG_FILE="/data/soft/huluer/redis/$SERVICE_NAME.conf"

    # Use awk to retrieve host, port from config file
    HOST=`awk '/^[[:blank:]]*bind/ { print $2 }' $CONFIG_FILE | tail -n1`
    PORT=`awk '/^[[:blank:]]*port/ { print $2 }' $CONFIG_FILE | tail -n1`
    PASS=`awk '/^[[:blank:]]*requirepass/ { print $2 }' $CONFIG_FILE | tail -n1`
    SOCK=`awk '/^[[:blank:]]*unixsocket\s/ { print $2 }' $CONFIG_FILE | tail -n1`

    # Just in case, use default host, port
    HOST=${HOST:-127.0.0.1}
    if [ "$SERVICE_NAME" = redis ]; then
        PORT=${PORT:-6379}
    else
        PORT=${PORT:-26739}
    fi

    # Setup additional parameters
    # e.g password-protected redis instances
    [ -z "$PASS"  ] || ADDITIONAL_PARAMS="-a $PASS"

    # shutdown the service properly
    if [ -e "$SOCK" ] ; then
    	$REDIS_CLI -s $SOCK $ADDITIONAL_PARAMS shutdown
    else
    	$REDIS_CLI -h $HOST -p $PORT $ADDITIONAL_PARAMS shutdown
    fi
    ```
*   设置权限

    ```bash
    chmod +x /data/soft/huluer/redis/bin/redis-shutdown
    ```
*   创建`huluer-redis.service`文件

    ```bash
    echo '
    [Unit]
    Description=Redis persistent key-value database
    After=network.target
    After=network-online.target
    Wants=network-online.target

    [Service]
    ExecStart=/data/soft/huluer/redis/bin/redis-server /data/soft/huluer/redis/redis.conf --supervised systemd
    ExecStop=/data/soft/huluer/redis/bin/redis-shutdown
    Type=notify
    User=huluer
    Group=huluer
    RuntimeDirectory=huluer
    RuntimeDirectoryMode=0755

    [Install]
    WantedBy=multi-user.target

    ' > /data/soft/huluer/redis/huluer-redis.service
    ```
*   注册系统服务

    ```bash
    sh -c 'cp -f /data/soft/huluer/redis/huluer-redis.service /usr/lib/systemd/system/huluer-redis.service'
    systemctl daemon-reload
    ```

## 配置 celery

*   创建`huluer-celery.srvice`文件

    ```bash
    echo '
    [Unit]
    Description=HuLuEr Celery Service
    After=network.target

    [Service]
    Type=simple
    PIDFile=/data/soft/huluer/celery/celery.pid
    ExecStartPre=/usr/bin/rm -f /data/soft/huluer/celery/celery.pid
    ExecStart=/data/soft/huluer/python3.6/bin/python3.6 /data/soft/huluer/python3.6/bin/celery worker --gid huluer --uid huluer --workdir /data/soft/huluer/backend --app celery_tasks.main --pool solo --pidfile /data/soft/huluer/celery/celery.pid --loglevel info --logfile /data/soft/huluer/celery/celery.log
    ExecReload=/bin/kill -HUP $MAINPID
    KillSignal=SIGQUIT
    PrivateTmp=true

    [Install]
    WantedBy=multi-user.target

    ' > /data/soft/huluer/celery/huluer-celery.service
    ```
*   注册为系统服务

    ```bash
    sh -c 'cp -f /data/soft/huluer/celery/huluer-celery.service /usr/lib/systemd/system/huluer-celery.service'
    systemctl daemon-reload
    ```

## 配置 uwsgi

*   安装`uWSGI`

    ```bash
    /data/soft/huluer/python3.6/bin/pip3.6 install uWSGI==2.0.18
    ```
*   创建`huluer.ini`文件

    ```bash
    echo '
    [uwsgi]
    uid = huluer
    gid = huluer
    pidfile = /data/soft/huluer/uwsgi/uwsgi.pid
    stats = /data/soft/huluer/uwsgi/stats.sock
    socket = 127.0.0.1:7776
    chdir = /data/soft/huluer/backend/
    wsgi-file = myhorizon/wsgi.py
    processes = 4
    threads = 2
    virtualenv=/data/soft/huluer/python3.6
    daemonize = /data/soft/huluer/uwsgi/uwsgi.log

    ' > /data/soft/huluer/uwsgi/huluer.ini
    ```
*   创建`huluer-uwsgi.service`文件

    ```bash
    echo '
    [Unit]
    Description=HuLuEr uWSGI Service
    After=syslog.target

    [Service]
    Type=forking
    PIDFile=/data/soft/huluer/uwsgi/uwsgi.pid
    ExecStart=/data/soft/huluer/python3.6/bin/uwsgi --ini /data/soft/huluer/uwsgi/huluer.ini
    ExecReload=/bin/kill -HUP $MAINPID
    KillSignal=SIGQUIT
    TimeoutStopSec=5
    KillMode=mixed
    PrivateTmp=true

    [Install]
    WantedBy=multi-user.target

    ' > /data/soft/huluer/uwsgi/huluer-uwsgi.service
    ```
*   注册系统服务

    ```bash
    sh -c 'cp -f /data/soft/huluer/uwsgi/huluer-uwsgi.service /usr/lib/systemd/system/huluer-uwsgi.service'
    systemctl daemon-reload
    ```

## 打包

*   创建安装脚本

    ```bash
    echo '
    #!/bin/bash
    set -e
    echo '关闭SELinux'
    setenforce 0
    sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux
    echo '创建葫芦儿用户'
    getent group huluer > /dev/null || groupadd -r huluer
    getent passwd huluer > /dev/null || useradd -r -d /data/soft/huluer -g huluer -s /sbin/nologin huluer
    echo '修改部署目录所有者'
    chown -R huluer:huluer /data/soft/huluer
    echo '安装依赖包'
    yum install -y gcc-c++ valgrind-devel systemtap-sdt-devel \
    bzip2-devel ncurses-devel gdbm-devel sqlite-devel \
    openssl-devel readline-devel zlib-devel xz-devel tk-devel \
    gperftools-devel openssl-devel pcre-devel zlib-devel \
    GeoIP-devel gd-devel perl-devel perl-ExtUtils-Embed libxslt-devel
    echo '注册为系统服务'
    cp -f /data/soft/huluer/nginx/huluer-nginx.service /usr/lib/systemd/system/huluer-nginx.service
    cp -f /data/soft/huluer/redis/huluer-redis.service /usr/lib/systemd/system/huluer-redis.service
    cp -f /data/soft/huluer/celery/huluer-celery.service /usr/lib/systemd/system/huluer-celery.service
    cp -f /data/soft/huluer/uwsgi/huluer-uwsgi.service /usr/lib/systemd/system/huluer-uwsgi.service
    systemctl daemon-reload
    echo '防火墙放行'
    firewall-cmd --add-port=80/tcp --add-port=7777/tcp
    firewall-cmd --add-port=80/tcp --add-port=7777/tcp --permanent

    ' > /data/soft/huluer/install.sh

    chmod +x /data/soft/huluer/install.sh
    ```
*   压缩

    ```bash
    cd /data/soft/
    tar czvf huluer.tar.gz huluer/{python3.6,nginx,redis,uwsgi,celery,backend,frontend,install.sh}
    ```
