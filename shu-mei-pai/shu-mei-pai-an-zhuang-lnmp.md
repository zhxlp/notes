# 树莓派安装 LNMP

- 安装软件包

  ```bash
  apt install -y nginx mariadb-server php7.3-fpm php7.3-cli  php7.3-mysql
  ```

- 配置 nginx

  ```properties
  server {
    listen       80 default_server;
    server_name  _;
    root         /data/www/demo;
    index        index.html index.htm index.php;
    location ~ \.php$ {
        try_files $uri =404;
        fastcgi_intercept_errors on;
        fastcgi_index  index.php;
        include        fastcgi_params;
        fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
        fastcgi_pass   unix:/run/php/php7.3-fpm.sock;
    }
  }
  ```

- 更改文件夹所有者

  ```bash
  chown -R www-data:www-data /data/www/demo
  ```
