# CentOS7 安装 MongoDB

## 方法 1：RPM 包安装

*   访问官网页面下载 rpm 包

    [https://www.mongodb.com/download-center/community](https://www.mongodb.com/download-center/community)

    ```bash
    # mongodb服务端
    wget https://repo.mongodb.org/yum/redhat/7/mongodb-org/3.6/x86_64/RPMS/mongodb-org-server-3.6.14-1.el7.x86_64.rpm
    # mongodb 客户端连接工具 mongo
    wget https://repo.mongodb.org/yum/redhat/7/mongodb-org/3.6/x86_64/RPMS/mongodb-org-shell-3.6.14-1.el7.x86_64.rpm
    # mongodb 工具集合：mongoimport, bsondump, mongodump, mongoexport, mongofiles, mongorestore, mongostat, mongotop
    wget https://repo.mongodb.org/yum/redhat/7/mongodb-org/3.6/x86_64/RPMS/mongodb-org-tools-3.6.14-1.el7.x86_64.rpm
    ```
*   安装 mongodb server

    ```bash
    yum install mongodb-org-server-3.6.14-1.el7.x86_64.rpm mongodb-org-tools-3.6.14-1.el7.x86_64.rpm mongodb-org-tools-3.6.14-1.el7.x86_64.rpm
    ```
*   设置远程访问

    修改`/etc/mongod.conf`文件`bindIP`部分

    ```bash
    net:
      port: 27017
      bindIp: 0.0.0.0
    ```
*   启动服务并开机自启

    ```bash
    systemctl restart mongod.service
    systemctl status mongod.service
    systemctl enable mongod.service
    ```

## 方法 2：两进制文件安装

*   下载二进制文件

    ```bash
    wget https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-rhel70-3.6.14.tgz
    ```
*   安装依赖

    ```bash
    yum install libcurl openssl
    ```
*   创建 mongo 用户和组

    ```bash
    /usr/sbin/groupadd -r mongod
    /usr/sbin/useradd -M -r -g mongod -d /var/lib/mongo -s /bin/false -c mongod mongod
    ```
*   解压文件

    ```bash
    tar xvf mongodb-linux-x86_64-rhel70-3.6.14.tgz
    mv mongodb-linux-x86_64-rhel70-3.6.14 /opt/mongodb
    ```
*   配置环境变量

    ```bash
    echo '
    # MongoDB
    export PATH=$PATH:/opt/mongodb/bin
    ' >> /etc/profile
    source /etc/profile
    ```
*   创建配置文件

    创建`/etc/mongod.conf`文件,内容如下

    ```properties
    # where to write logging data.
    systemLog:
      destination: file
      logAppend: true
      path: /var/log/mongodb/mongod.log

    # Where and how to store data.
    storage:
      dbPath: /var/lib/mongo
      journal:
        enabled: true

    # how the process runs
    processManagement:
      fork: true  # fork and run in background
      pidFilePath: /var/run/mongodb/mongod.pid  # location of pidfile
      timeZoneInfo: /usr/share/zoneinfo

    # network interfaces
    net:
      port: 27017
      bindIp: 0.0.0.0
    ```
*   创建 log 目录和 dbPath 目录

    ```bash
    # log
    mkdir -p /var/log/mongodb/
    chown -R mongod:mongod /var/log/mongodb/
    # dbPath
    mkdir -p /var/lib/mongo
    chown -R mongod:mongod /var/lib/mongo
    ```
*   注册系统服务

    创建`/lib/systemd/system/mongod.service`文件，内容如下

    ```properties
    [Unit]
    Description=MongoDB Database Server
    Documentation=https://docs.mongodb.org/manual
    After=network.target

    [Service]
    User=mongod
    Group=mongod
    Environment="OPTIONS=-f /etc/mongod.conf"
    ExecStart=/opt/mongodb/bin/mongod $OPTIONS
    ExecStartPre=/usr/bin/mkdir -p /var/run/mongodb
    ExecStartPre=/usr/bin/chown mongod:mongod /var/run/mongodb
    ExecStartPre=/usr/bin/chmod 0755 /var/run/mongodb
    PermissionsStartOnly=true
    PIDFile=/var/run/mongodb/mongod.pid
    Type=forking
    # file size
    LimitFSIZE=infinity
    # cpu time
    LimitCPU=infinity
    # virtual memory size
    LimitAS=infinity
    # open files
    LimitNOFILE=64000
    # processes/threads
    LimitNPROC=64000
    # locked memory
    LimitMEMLOCK=infinity
    # total threads (user+kernel)
    TasksMax=infinity
    TasksAccounting=false
    # Recommended limits for for mongod as specified in
    # http://docs.mongodb.org/manual/reference/ulimit/#recommended-settings

    [Install]
    WantedBy=multi-user.target
    ```
*   重新加载 systemd

    ```bash
    systemctl daemon-reload
    ```
*   启动服务并设置开机自启

    ```
    systemctl restart mongod.service
    systemctl status mongod.service
    systemctl enable mongod.service
    ```
