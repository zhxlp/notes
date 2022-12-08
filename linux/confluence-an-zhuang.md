# confluence 安装

## 参考链接

[破解](https://zhile.io/2018/12/20/atlassian-license-crack.html)

[confluence 官网](https://www.atlassian.com/software/confluence)

## 安装

*   环境

    操作系统：CentOS 7.6.1810

    数据库：MySQL 5.7.26

    JDK：1.8\_201

    Confluence：7.1.0
*   安装字体库

    ```bash
    yum install -y fontconfig
    ```
* 安装并配置 JDK
* 安装 MySQL
*   配置 MySQL

    在`/etc/my.cnf`文件添加如下内容

    ```properties
    [mysqld]
    character-set-server=utf8
    collation-server=utf8_bin
    default-storage-engine=INNODB
    max_allowed_packet=256M
    innodb_log_file_size=2GB
    transaction-isolation=READ-COMMITTED
    binlog_format=row
    ```
*   创建数据库及用户

    ```sql
    CREATE DATABASE confluence CHARACTER SET utf8 COLLATE utf8_bin;
    GRANT ALL PRIVILEGES ON confluence.* TO 'confluence'@'localhost' IDENTIFIED BY 'redhat';
    ```
*   下载软件包并解压

    ```bash
    wget https://product-downloads.atlassian.com/software/confluence/downloads/atlassian-confluence-7.1.0.tar.gz
    mkdir -p /opt/atlassian
    tar -xvf atlassian-confluence-7.1.0.tar.gz -C /opt/atlassian
    cd /opt/atlassian
    mv atlassian-confluence-7.1.0 confluence
    ```
*   下载 JDBC Jar 包

    ```bash
    wget https://repo1.maven.org/maven2/mysql/mysql-connector-java/5.1.48/mysql-connector-java-5.1.48.jar
    mv mysql-connector-java-5.1.48.jar /opt/atlassian/confluence/confluence/WEB-INF/lib/
    ```
*   创建运行用户

    ```bash
    useradd --create-home --comment "Account for running Confluence" --shell /bin/bash confluence
    ```
*   创建并配置`confluence.home`目录

    ```bash
    mkdir -p /var/atlassian/application-data/confluence
    ```

    修改`/opt/atlassian/confluence/confluence/WEB-INF/classes/confluence-init.properties`文件，添加如下内容

    ```properties
    confluence.home=/var/atlassian/application-data/confluence
    ```
*   配置运行用户

    编辑`/opt/atlassian/confluence/bin/user.sh`，修改如下内容

    ```properties
    CONF_USER="confluence"
    ```
*   修改文件权限

    ```bash
    chown -R confluence:confluence /opt/atlassian/confluence
    chmod -R u=rwx,go-rwx /opt/atlassian/confluence
    chown -R confluence:confluence /var/atlassian/application-data/confluence
    chmod -R u=rwx,go-rwx /var/atlassian/application-data/confluence
    ```
*   创建启动脚本

    编辑`/etc/init.d/confluence`

    ```properties
    #!/bin/bash

    # Confluence Linux service controller script
    cd "/opt/atlassian/confluence/bin"

    case "$1" in
        start)
            ./start-confluence.sh
            ;;
        stop)
            ./stop-confluence.sh
            ;;
        restart)
            ./stop-confluence.sh
            ./start-confluence.sh
            ;;
        *)
            echo "Usage: $0 {start|stop|restart}"
            exit 1
            ;;
    esac
    ```

    添加可执行权限

    ```bash
    chmod +x /etc/init.d/confluence
    ```

## 破解

* 下载破解程序`atlassian-agent.jar`到`/opt/atlassian/`目录
*   配置破解程序

    修改`/opt/atlassian/confluence/bin/setenv.sh`添加如下内容

    ```properties
    export JAVA_OPTS="-javaagent:/opt/atlassian/atlassian-agent.jar ${JAVA_OPTS}"
    ```
*   获取许可

    ```bash
    cd /opt/atlassian/
    # Confluence
    java -jar atlassian-agent.jar -p conf -m test@test.com -n test -o test -s BYH6-K52J-SZX1-MEJV
    # Confluence Team Calendars
    java -jar atlassian-agent.jar -p tc -m test@test.com -n test -o test -s BYH6-K52J-SZX1-MEJV
    # Confluence Questions
    java -jar atlassian-agent.jar -p questions -m test@test.com -n test -o test -s BYH6-K52J-SZX1-MEJV
    ```

## 恢复模式

修改`/opt/atlassian/confluence/bin/setenv.sh`文件如下内容

```properties
# recovery_admin 用户密码
CATALINA_OPTS="-Datlassian.recovery.password=redhat${CATALINA_OPTS}"
```

重启服务
