# Docker 安装 guacamole

- 拉取镜像

  ```bash
  docker pull guacamole/guacamole:1.1.0
  docker pull guacamole/guacd:1.1.0
  docker pull mysql:5.7.29
  ```

- 运行**guacamole/guacd**

  提供从已发布的 guacamole-server 源构建的 guacd 守护程序， 并支持 VNC，RDP，SSH，telnet 和 Kubernetes

  ```bash
  docker run --name some-guacd -d -p 4822:4822 --restart=always guacamole/guacd:1.1.0
  ```

- 导出数据库初始化语句

  ```bash
  docker run --rm guacamole/guacamole:1.1.0 /opt/guacamole/bin/initdb.sh --mysql > initdb.sql
  ```

- 运行 mysql

  ```bash
  docker run --name some-mysql \
      -p 3306:3306 \
  	-e MYSQL_ROOT_PASSWORD='guacamole_pass' \
  	-e MYSQL_DATABASE='guacamole_db' \
  	-e MYSQL_USER='guacamole_user' \
  	-e MYSQL_PASSWORD='guacamole_pass' \
  	-v mysql:/var/lib/mysql \
  	--restart=always -d mysql:5.7.29
  ```

- 导入初始化数据库

  ```bash
  docker exec -i some-mysql sh -c 'exec mysql -u"$MYSQL_USER" -p"$MYSQL_PASSWORD" -D"$MYSQL_DATABASE"' < initdb.sql
  ```

- 运行**guacamole/guacamole**

  为在 Tomcat 8 中运行的 Guacamole Web 应用程序提供对 WebSocket 的支持。当镜像基于 Docker 链接或环境变量启动时，将自动生成连接到 guacd，MySQL，PostgreSQL，LDAP 等所需的配置

  ```bash
  docker run --name some-guacamole \
      --link some-guacd:guacd        \
      --link some-mysql:mysql \
  	-e MYSQL_DATABASE='guacamole_db' \
  	-e MYSQL_USER='guacamole_user' \
  	-e MYSQL_PASSWORD='guacamole_pass' \
  	-e GUACAMOLE_HOME=/etc/guacamole \
  	-v /etc/guacamole:/etc/guacamole \
  	--restart=always -d -p 8080:8080 guacamole/guacamole:1.1.0
  ```

- 登录

  http://_`HOSTNAME`_:8080/guacamole/

  username:guacadmin

  password:guacadmin
