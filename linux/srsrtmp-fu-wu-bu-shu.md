# SRS-RTMP 服务部署

*   环境

    Ubuntu 18.04
*   下载源码包

    ```bash
    wget https://github.com/ossrs/srs/archive/v2.0-r6.tar.gz
    ```
*   编译安装

    ```bash
    # 解压
    tar xvf srs-2.0-r6.tar.gz
    # 进入编译目录
    cd srs-2.0-r6/trunk/
    # 编译安装
    ./configure && make && make install
    ```
*   创建 rtmp 服务配置文件

    ```bash
    mkdir /etc/srs
    echo '
    listen              1935;
    max_connections     1000;
    daemon              off;
    pid                 /usr/local/srs/objs/srs.pid;
    srs_log_tank        file;
    srs_log_file        /var/log/srs.log;
    srs_log_level       trace;
    vhost __defaultVhost__ {
    }

    ' > /etc/srs/rtmp.conf
    ```
*   创建服务

    ```bash
    echo '[Unit]
    Description=SRS RTMP Server
    After=syslog.target network.target

    [Service]
    Type=simple
    User=root
    ExecStart=/usr/local/srs/objs/srs -c /etc/srs/rtmp.conf
    Restart=on-failure

    [Install]
    WantedBy=multi-user.target
    ' > /etc/systemd/system/srs-rtmp.service

    systemctl daemon-reload
    ```
*   启动服务并设置开机自启

    ```bash
    systemctl restart srs-rtmp.service
    systemctl enable srs-rtmp.service
    ```
