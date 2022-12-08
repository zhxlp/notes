# jira 安装

## 参考链接

[破解](https://zhile.io/2018/12/20/atlassian-license-crack.html)

[jira 官网](https://www.atlassian.com/software/jira)

## 安装

*   环境

    操作系统：CentOS 7.6.1810

    数据库：MySQL 5.7.26

    JDK：1.8\_201

    Jira：8.5.1
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
    default-storage-engine=INNODB
    character_set_server=utf8mb4
    innodb_default_row_format=DYNAMIC
    innodb_large_prefix=ON
    innodb_file_format=Barracuda
    innodb_log_file_size=2G
    ```
*   创建数据库及用户

    ```sql
    CREATE DATABASE jira CHARACTER SET utf8mb4 COLLATE utf8mb4_bin;
    GRANT ALL PRIVILEGES on jira.* TO 'jira'@'localhost' IDENTIFIED BY 'redhat';
    flush privileges;
    ```
*   下载软件包并解压

    ```bash
    wget https://product-downloads.atlassian.com/software/jira/downloads/atlassian-jira-software-8.5.1.tar.gz
    mkdir -p /opt/atlassian
    tar -xvf atlassian-jira-software-8.5.1.tar.gz -C /opt/atlassian
    cd /opt/atlassian
    mv atlassian-jira-software-8.5.1-standalone jira
    ```
*   下载 JDBC Jar 包

    ```bash
    wget https://repo1.maven.org/maven2/mysql/mysql-connector-java/5.1.48/mysql-connector-java-5.1.48.jar
    mv mysql-connector-java-5.1.48.jar /opt/atlassian/jira/lib
    ```
*   创建运行用户

    ```bash
    useradd --create-home --comment "Account for running Jira Software" --shell /bin/bash jira
    ```
*   创建并配置`jira.home`目录

    ```bash
    mkdir -p /var/atlassian/application-data/jira
    ```

    修改`/opt/atlassian/jira/atlassian-jira/WEB-INF/classes/jira-application.properties`文件，添加如下内容

    ```properties
    jira.home = /var/atlassian/application-data/jira
    ```
*   配置运行用户

    编辑`/opt/atlassian/jira/bin/user.sh`，修改如下内容

    ```properties
    CONF_USER="jira"
    ```
*   修改文件权限

    ```bash
    chown -R jira:jira /opt/atlassian/jira
    chmod -R u=rwx,go-rwx /opt/atlassian/jira
    chown -R jira:jira /var/atlassian/application-data/jira
    chmod -R u=rwx,go-rwx /var/atlassian/application-data/jira
    ```
*   创建启动脚本

    编辑`/etc/init.d/jira`

    ```properties
    #!/bin/bash

    # JIRA Linux service controller script
    cd "/opt/atlassian/jira/bin"

    case "$1" in
        start)
            ./start-jira.sh
            ;;
        stop)
            ./stop-jira.sh
            ;;
        *)
            echo "Usage: $0 {start|stop}"
            exit 1
            ;;
    esac
    ```

    添加可执行权限

    ```bash
    chmod +x /etc/init.d/jira
    ```

## 破解

* 下载破解程序`atlassian-agent.jar`到`/opt/atlassian/`目录
*   配置破解程序

    修改`/opt/atlassian/jira/bin/setenv.sh`添加如下内容

    ```properties
    export JAVA_OPTS="-javaagent:/opt/atlassian/atlassian-agent.jar ${JAVA_OPTS}"
    ```
*   获取许可

    ```bash
    cd /opt/atlassian/
    # Jira Software
    java -jar atlassian-agent.jar -p jira -m test@test.com -n test -o test -s BTQ1-ISAR-VBIS-1OBO
    # Jira Core
    java -jar atlassian-agent.jar -p jc -m test@test.com -n test -o test -s BTQ1-ISAR-VBIS-1OBO
    ```
