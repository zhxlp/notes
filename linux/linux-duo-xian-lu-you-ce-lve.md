# Linux 多线路由策略

## 问题

服务器有两张网卡`eth0` `10.0.10.10/24` 和`eth1` `10.0.20.10/24`,默认路由`eth0`,外部网络访问服务器时，只能访问`eth0`，无法访问`eth1`，通过网络抓包，分析结果为：访问`eth1`时，服务器正常接收到数据包，服务器响应数据时，却从`eth0`接口响应，IP 地址不一致，导致响应失败，客户端无法接受到服务器的正常响应。

## 解决思路

服务器从那一个网络接口接收数据，就从哪一个网络接口响应数据。

## 实现方法

*   定义路由表

    编辑`/etc/iproute2/rt_tables`文件，根据文件格式，添加如下内容

    ```properties
    # id(不重复)    路由表名称(不重复)
    1               net_10_0_10_0
    2               net_10_0_20_0
    ```
*   设置路由

    ```bash
    # 添加默认路由
    ip route add default via 10.0.10.1 dev eth0
    # 清空路由表net_10_0_10_0
    ip route flush table net_10_0_10_0
    # 在net_10_0_10_0路由表，添加路由，访问10.0.10.0/24网段，使用eth0接口的10.0.10.10IP
    ip route add 10.0.10.0/24 dev eth0 src 10.0.10.10 table net_10_0_10_0
    # 在net_10_0_10_0路由表，设备默认路由为10.0.10.1
    ip route add default via 10.0.10.1 table net_10_0_10_0
    # 删除路由策略
    ip rule del from 10.0.10.10 table net_10_0_10_0
    # 添加路由策略，来自 10.0.10.10的数据包使用net_10_0_10_0路由表
    ip rule add from 10.0.10.10 table net_10_0_10_0

    ip route flush table net_10_0_20_0
    ip route add 10.0.20.0/24 dev eth1 src 10.0.20.10 table net_10_0_20_0
    ip route add default via 10.0.20.1 table net_10_0_20_0
    ip rule del from 10.0.20.10 table net_10_0_20_0
    ip rule add from 10.0.20.10 table net_10_0_20_0
    ```
*   定义网络重启自动添加路由

    *   创建`/etc/init.d/network-route`脚本,实现执行脚本添加路由

        ```bash
        echo '#!/bin/bash
        ip route add default via 10.0.10.1 dev eth0
        ip route flush table net_10_0_10_0
        ip route add 10.0.10.0/24 dev eth0 src 10.0.10.10 table net_10_0_10_0
        ip route add default via 10.0.10.1 table net_10_0_10_0
        ip rule del from 10.0.10.10 table net_10_0_10_0
        ip rule add from 10.0.10.10 table net_10_0_10_0

        ip route flush table net_10_0_20_0
        ip route add 10.0.20.0/24 dev eth1 src 10.0.20.10 table net_10_0_20_0
        ip route add default via 10.0.20.1 table net_10_0_20_0
        ip rule del from 10.0.20.10 table net_10_0_20_0
        ip rule add from 10.0.20.10 table net_10_0_20_0
        ' > /etc/init.d/network-route
        # 添加执行权限
        chmod +x /etc/init.d/network-route
        ```

    ```
    ```
*   添加脚本到网络服务，实现启动和重启网络时，自动执行脚本

    编辑`/etc/init.d/network`文件，在`start`部分添加如下内容

    ```properties
    start)
    .....
        bash /etc/init.d/network-route
        ;;
    .....
    ```

    * 重启网络

    ```bash
    systemctl daemon-reload
    systemctl restart network.service
    ```

```
```
