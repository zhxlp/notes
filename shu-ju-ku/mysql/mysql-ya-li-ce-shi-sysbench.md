# MySql 压力测试-sysbench

*   下载源码包

    访问[sysbench 发布页面](https://github.com/akopytov/sysbench/releases)下载源码包
*   安装依赖

    ```bash
    yum -y install make automake libtool pkgconfig libaio-devel mariadb-devel openssl-devel
    ```
*   编译安装

    ```bash
    ./autogen.sh
    ./configure
    make -j
    make install
    ```
*   参数说明

    ```bash
    --threads=32  # 工作线程数
    --events=0 # 请求总数限制
    --time=3600  # 限制总执行时间
    --report-interval=60 # 定期报告指定间隔时间内的统计信息
    --db-driver=mysql  # 数据库驱动
    --mysql-table-engine=innodb  # 数据库引擎
    --mysql-host=10.0.10.30  # 数据库地址
    --mysql-port=3306  # 数据库端口
    --mysql-user=root # 数据库用户名
    --mysql-password='qwe123.'  # 数据库密码
    --mysql-db=sbtest  # 数据库名
    --oltp-tables-count=10  # 每次测试生成表数量
    --oltp-table-size=10000  # 每张表填充数据量
    /usr/local/share/sysbench/tests/include/oltp_legacy/oltp.lua # 指定测试脚本
    prepare # prepare准备环境,run开始测试,cleanup清理环境
    ```
*   准备环境

    ```bash
    sysbench --threads=32  \
    --events=0 \
    --time=3600 \
    --report-interval=60 \
    --db-driver=mysql \
    --mysql-table-engine=innodb \
    --mysql-host=10.0.100.61 \
    --mysql-port=3306 \
    --mysql-user=root \
    --mysql-password='qwe123.' \
    --mysql-db=sbtest \
    --oltp-tables-count=10 \
    --oltp-table-size=10000 \
    /usr/local/share/sysbench/tests/include/oltp_legacy/oltp.lua \
    prepare
    ```
*   测试

    ```bash
    sysbench --threads=32  \
    --events=0 \
    --time=3600 \
    --report-interval=60 \
    --db-driver=mysql \
    --mysql-table-engine=innodb \
    --mysql-host=10.0.100.61 \
    --mysql-port=3306 \
    --mysql-user=root \
    --mysql-password='qwe123.' \
    --mysql-db=sbtest \
    --oltp-tables-count=10 \
    --oltp-table-size=10000 \
    /usr/local/share/sysbench/tests/include/oltp_legacy/oltp.lua \
    run
    ```
*   清理环境

    ```bash
    sysbench --threads=32  \
    --events=0 \
    --time=3600 \
    --report-interval=60 \
    --db-driver=mysql \
    --mysql-table-engine=innodb \
    --mysql-host=10.0.100.61 \
    --mysql-port=3306 \
    --mysql-user=root \
    --mysql-password='qwe123.' \
    --mysql-db=sbtest \
    --oltp-tables-count=10 \
    --oltp-table-size=10000 \
    /usr/local/share/sysbench/tests/include/oltp_legacy/oltp.lua \
    cleanup
    ```
*   测试结果

    > TPS Transactions Per Second，即数据库每秒执行的事务数，以 commit 成功次数为准。
    >
    > QPS Queries Per Second，即数据库每秒执行的 SQL 数（含 insert、select、update、delete 等）。
