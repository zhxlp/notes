# GuacamoleServer 编译安装

## 环境

操作系统：CentOS7.6

## 安装

- 安装依赖

  ```bash
  yum install -y cairo-devel libjpeg-turbo-devel libjpeg-devel libpng-devel libtool uuid-devel ffmpeg-devel freerdp-devel pango-devel libssh2-devel libtelnet-devel libvncserver-devel libwebsockets-devel pulseaudio-libs-devel openssl-devel libvorbis-devel libwebp-devel
  ```

- 下载和解压源码

  ```bash
  wget https://github.com/apache/guacamole-server/archive/1.1.0.tar.gz
  tar -xzf guacamole-server-1.1.0.tar.gz -C /usr/local/src/
  ```

- 生成 configure

  ```bash
  cd /usr/local/src/guacamole-server/
  autoreconf -fi
  ```

- 编译安装

  ```bash
  ./configure --with-init-dir=/etc/init.d --with-systemd-dir=/usr/lib/systemd/system
  make
  make install
  ```

- 启动服务

  ```bash
  systemctl restart guacd.service
  systemctl enable guacd.service
  ```
