# PHP Xdebug

参考连接：https://www.jetbrains.com/help/phpstorm/multiuser-debugging-via-xdebug-proxies.html

![DBGp proxy](../.gitbook/assets/schema-proxy.png)

## Xdebug

- 设置 xdebug 配置

  修改`/etc/php.d/15-xdebug.ini`

  ```properties
  zend_extension=xdebug.so
  xdebug.remote_enable = 1
  ; xdebug.remote_host=hostname_or_ip_of_the_dbgp_proxy_goes_here
  xdebug.remote_host = 127.0.0.1
  xdebug.remote_port = 9001
  ```

## DBGp 代理

- 从 Komodo 的[下载页面](http://code.activestate.com/komodo/remotedebugging/)中，我们可以找到适用于我们平台的 DBGp 代理的 Python 二进制文件

  ![image-20200516002006147](../.gitbook/assets/image-20200516002006147.png)

- 运行 DBGp 代理

  在 Web 服务器上或在可以与 Web 服务器和所有开发人员计算机进行通信的计算机上启动 DBGp 代理。DBGp 代理可执行文件接受两个参数：`-d`和`-i`。

  参数定义了从 Web 服务器监听调试器连接的 IP 地址和端口，以及监听开发人员的 IP 地址和端口。

  例如，监听环回地址（`127.0.0.1` ）和端口`9001上的调试器连接，并监计算机IP地址和端口`9002`上的开发人员。

  ```bash
  export PYTHONPATH=$PYTHONPATH:`pwd`/python3lib:`pwd`/pythonlib
  ./pydbgpproxy -d 127.0.0.1:9001 -i 10.8.0.1:9002
  ```

  ![image-20200516004018777](../.gitbook/assets/image-20200516004018777.png)

- 配置 IDE

  ![image-20200516013927850](../.gitbook/assets/image-20200516013927850.png)

- 配置浏览器

  ![image-20200516014040467](../.gitbook/assets/image-20200516014040467.png)
