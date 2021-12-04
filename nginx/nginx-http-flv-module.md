# nginx-http-flv-module

[nginx-http-flv-module](https://github.com/winshining/nginx-http-flv-module/blob/master/README.CN.md)是基于[nginx-rtmp-module](https://github.com/arut/nginx-rtmp-module)的流媒体服务器。

## 安装

- 安装依赖

  [CentOS7]

  ```bash
  yum install -y pcre pcre-devel zlib zlib-devel gcc-c++ openssl openssl-devel
  ```

  [Ubuntu]

  ```bash
  sudo apt-get install build-essential libtool libpcre3 libpcre3-dev zlib1g-dev openssl
  ```

- 编译安装

  ```bash
  ./configure --with-http_ssl_module --add-module=/path/to/nginx-http-flv-module
  make
  make install
  ```

## 配置

- `http-flv/http-flv.conf`

  ```properties
  server {
      # set according to your environment
      listen 8080;
      server_name localhost;

      location /live {
          flv_live on;
          chunked_transfer_encoding  on;

          add_header 'Access-Control-Allow-Origin' '*';
          add_header 'Access-Control-Allow-Credentials' 'true';
      }
  }

  ```

- `http-flv/rtmp.conf`

  ```properties
  #because the vhost feature is not perfect in multi-processes
  #mode yet, the directive worker_processes should be set to 1

  rtmp_auto_push on;
  rtmp_auto_push_reconnect 1s;
  rtmp_socket_dir /tmp;

  rtmp {
      out_queue    4096;
      out_cork     8;
      max_streams  256;
      timeout      15s;
      drop_idle_publisher 15s;

      server {
          # set according to your environment
          # listen 1935;
          # server_name localhost;

          application myapp {
              live on;
              gop_cache on;
          }

          application hls {
              live on;
              hls on;
              hls_path /tmp/hls;
          }

          application dash {
              live on;
              dash on;
              dash_path /tmp/dash;
          }
      }
  }

  ```

- `nginx.conf`

  ```properties
  .....
  include http-flv/rtmp.conf;
  ....
  http {
  ....
      include http-flv/http-flv.conf;
  ....
  }
  ```
