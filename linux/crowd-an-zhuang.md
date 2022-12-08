# crowd 安装

## 参考链接

[破解](https://zhile.io/2018/12/20/atlassian-license-crack.html)

[confluence 官网](https://www.atlassian.com/software/confluence)

## 安装

*   环境

    操作系统：CentOS 7.6.1810

    数据库：MySQL 5.7.26

    JDK：1.8\_201

    Crowd：3.7.0
*   安装字体库

    ```bash
    yum install -y fontconfig
    ```
* 安装并配置 JDK
* 安装 MySQL
* 配置 MySQL
*   创建数据库及用户

    ```sql
    CREATE DATABASE crowd CHARACTER SET utf8 COLLATE utf8_bin;
    GRANT ALL PRIVILEGES ON crowd.* TO 'crowd'@'localhost' IDENTIFIED BY 'redhat';
    ```
*   下载软件包并解压

    ```bash
    wget https://product-downloads.atlassian.com/software/crowd/downloads/atlassian-crowd-3.7.0.tar.gz
    mkdir -p /opt/atlassian
    tar -xvf atlassian-crowd-3.7.0.tar.gz -C /opt/atlassian
    cd /opt/atlassian
    mv atlassian-crowd-3.7.0 crowd
    ```
*   下载 JDBC Jar 包

    ```bash
    wget https://repo1.maven.org/maven2/mysql/mysql-connector-java/5.1.48/mysql-connector-java-5.1.48.jar
    mv mysql-connector-java-5.1.48.jar /opt/atlassian/crowd/crowd-webapp/WEB-INF/lib/
    ```
*   创建并配置`crowd.home`目录

    ```bash
    mkdir -p /var/atlassian/application-data/crowd
    ```

    修改`/opt/atlassian/crowd/crowd-webapp/WEB-INF/classes/crowd-init.properties`文件，添加如下内容

    ```properties
    crowd.home=/var/atlassian/application-data/crowd
    ```
*   创建启动脚本

    编辑`/etc/init.d/crowd`

    ```properties
    #!/bin/bash

    # Crowd Linux service controller script
    cd "/opt/atlassian/crowd"

    case "$1" in
        start)
            ./start_crowd.sh
            ;;
        stop)
            ./stop_crowd.sh
            ;;
        *)
            echo "Usage: $0 {start|stop}"
            exit 1
            ;;
    esac
    ```

    添加可执行权限

    ```bash
    chmod +x /etc/init.d/crowd
    ```

## 破解

* 下载破解程序`atlassian-agent.jar`到`/opt/atlassian/`目录
*   配置破解程序

    修改`/opt/atlassian/crowd/apache-tomcat/bin/setenv.sh`添加如下内容

    ```properties
    export JAVA_OPTS="-javaagent:/opt/atlassian/atlassian-agent.jar ${JAVA_OPTS}"
    ```
*   获取许可

    ```bash
    cd /opt/atlassian/
    # crowd
    java -jar atlassian-agent.jar -p crowd -m test@test.com -n test -o test -s BYH6-K52J-SZX1-MEJV
    ```
