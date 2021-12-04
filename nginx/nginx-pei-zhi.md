# Nginx 配置

## 静态服务器

```properties
server {
  listen 80;
  server_name b.test.com;
  root /data/www/test/;
  index index.html index.htm;

}
```

## 反向代理

```properties
server {
  listen 80;
  server_name b.test.com;

  location / {
    proxy_set_header  Host  $host;
    proxy_set_header  X-Real-IP  $remote_addr;
    proxy_set_header  X-Forwarded-For  $proxy_add_x_forwarded_for;
    proxy_pass http://10.0.100.63;
  }
}
```

- `Host`：客户端访问的主机地址

- `X-Real-IP`：用于记录真实客户端 IP(发送请求的真实 IP，多级代理，即上一级代理)

- `X-Forwarded-For`：是用于记录代理信息，每经过一级代理(匿名代理除外)的 IP

格式：client, proxy1, proxy2

## 负载均衡

```properties
upstream test {
  server 192.168.2.10:8000 weight=5;
  server 192.168.2.11:8000 weight=5 max_conns=800;
  server 192.168.2.12:8000 weight=5 max_fails=1  fail_timeout=30s;
}
```

- `weight`：权重，值越大，访问的比例越大
- `max_conns`：最大连接数
- `max_fails`：最大失败次数，与`fail_timeout`配合使用
- `fail_timeout`：失败检测周期，例：30s 内失败 1 次，将 server 标记为 down,等待 30s 不接受请求，30s 再接收请求进行检测
- `down`：表示单前的 server 暂时不参与负载
- `backup`：备用 server，其它 server 繁忙或者 down 时，请求此 server

### 算法

- 轮询(默认)

  1:1 处理请求

  ```properties
  upstream test {
    server 192.168.2.10:8000;
    server 192.168.2.11:8000;
  }
  ```

- 权重

  根据权重值比例处理请求,值越大，请求比例越大

  ```properties
  upstream test {
    server 192.168.2.10:8000 weight=1;
    server 192.168.2.11:8000 weight=2;
  }
  ```

- ip\_哈希算法

  每个请求按访问 ip 的 hash 结果分配，这样每个访客固定访问一个应用服务器，可以解决 session 共享的问题。

  ```properties
  upstream test {
    ip_hash;
    server 192.168.2.10:8000;
    server 192.168.2.11:8000;
  }
  ```

- least_conn 最小连接

  比较每个后端的 conns/weight，选取该值最小的后端。

- least_time 最小响应时间

## SSL

```properties
server {
  listen 443;
  server_name b.test.com;
  ssl on;
  root /data/www/test/;
  index index.html index.htm;
  ssl_certificate   /etc/nginx/ssl/server.crt;
  ssl_certificate_key  /etc/nginx/ssl/server.key;
  ssl_session_timeout 5m;
  ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
  ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
  ssl_prefer_server_ciphers on;

}

# 配置http跳转https
server {
  listen 80;
  server_name b.test.com;
  rewrite ^(.*)$ https://$host$1 permanent;
}
```

## PHP-FPM

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
      fastcgi_pass   127.0.0.1:9000;
  }
}
```

## 缓存

```properties
location ~ .*\.(gif|jpg|jpeg|png|bmp|swf)$ {
  expires 30d;
  access_log off;
}

location ~ .*\.(js|css)?$ {
  expires 12h;
  access_log off;
}
```

## 正则规则

- `=`：精确配置
- `~`：区分大小写匹配
- `~*`：不区分大小写匹配
- `^`：匹配字符串的开始位置
- `$`：匹配字符串的结束位置
- `.`：匹配除换行符以外的任意字符
- `[a-z]`：匹配 a~z 的任意单个字母
- `\d`：匹配数字
- `\w`：匹配字母、数字、下划线、汉字
- `\s`：匹配任意的空白符
- `(png|jpg|bmp)`：匹配 png 或 jpg 或 bmp
- `*`：重复 0 次或多次
- `?`：重复 0 次或 1 次
- `+`：重复 1 次以上
- `{n,m}`：重复 n~m 次
