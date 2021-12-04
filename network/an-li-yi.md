# 案例一

## 简介

### 网络信息

![网络拓扑图](../.gitbook/assets/Snipaste\_2020-08-02\_14-08-03.png)

* 区域一
  * VlanId：2
  * 网段：192.168.1.0/24
  * 网关：192.168.1.1
* 区域二
  * VlanId：3
  * 网段：192.168.2.0/24
  * 网关：192.168.2.1
* 区域三
  * VlanId：4
  * 网段：192.168.3.0/24
  * 网关：192.168.3.1
* 区域四
  * VlanId：1
  * 网段：192.168.100.0/24
  * 网关：192.168.100.1

### 要求

区域四与其它三个区域可以相互访问，但其它三个区域之间不能相互范围

## 步骤

*   配置 LSW1 交换机

    ```bash
    # 进入系统特权模式
    system-view

    # 创建vlan2 vlan3 vlan4
    vlan batch 2 to 4

    # 进入以太网接口一
    interface GigabitEthernet0/0/1
    # 设置接口连接方式为 access
    port link-type access
    # 设置接口为 vlan2
    port default vlan 2
    quit
    display current-configuration interface GigabitEthernet0/0/1

    interface GigabitEthernet0/0/2
    port link-type access
    port default vlan 3
    quit
    display current-configuration interface GigabitEthernet0/0/2

    interface GigabitEthernet0/0/3
    port link-type access
    port default vlan 4
    quit
    display current-configuration interface GigabitEthernet0/0/3

    # 开启交换机的dhcp功能
    dhcp enable

    # 进入vlan1 虚拟网络接口
    interface Vlanif 1
    # 设置IP地址作为网关使用
    ip address 192.168.100.1 255.255.255.0
    # 设置DHCP基于端口配置
    dhcp select interface
    # 配置DNS列表
    dhcp server dns-list 223.5.5.5
    dhcp server dns-list 8.8.8.8
    # 配置不参与DHCP分配的地址范围
    dhcp server excluded-ip-address 192.168.100.2 192.168.100.100
    # 配置租期为1天
    dhcp server lease day 1
    quit
    display current-configuration interface Vlanif 1

    interface Vlanif 2
    ip address 192.168.1.1 255.255.255.0
    dhcp select interface
    dhcp server dns-list 223.5.5.5
    dhcp server dns-list 8.8.8.8
    dhcp server excluded-ip-address 192.168.1.2 192.168.1.100
    dhcp server lease day 1
    quit
    display current-configuration interface Vlanif 2

    interface Vlanif 3
    ip address 192.168.2.1 255.255.255.0
    dhcp select interface
    dhcp server dns-list 223.5.5.5
    dhcp server dns-list 8.8.8.8
    dhcp server excluded-ip-address 192.168.2.2 192.168.2.100
    dhcp server lease day 1
    quit
    display current-configuration interface Vlanif 3

    interface Vlanif 4
    ip address 192.168.3.1 255.255.255.0
    dhcp select interface
    dhcp server dns-list 223.5.5.5
    dhcp server dns-list 8.8.8.8
    dhcp server excluded-ip-address 192.168.3.2 192.168.3.100
    dhcp server lease day 1
    quit
    display current-configuration interface Vlanif 4

    # 创建acl 3001 ,用于出口数据拦截
    acl 3001
    # 设置步长为 10
    step 10
    # 放行网关 192.168.1.1 发送给192.168.1.0/24 网段的数据
    rule permit ip source 192.168.1.1 0.0.0.0 destination 192.168.1.0 0.0.0.255
    # 放行网关 192.168.2.1 发送给192.168.2.0/24 网段的数据
    rule permit ip source 192.168.2.1 0.0.0.0 destination 192.168.2.0 0.0.0.255
    # 放行网关 192.168.3.1 发送给192.168.3.0/24 网段的数据
    rule permit ip source 192.168.3.1 0.0.0.0 destination 192.168.3.0 0.0.0.255
    # 拦截来至 192.168.1.0/24 网段的数据
    rule deny ip source 192.168.1.0 0.0.0.255 destination any
    # 拦截来至 192.168.2.0/24 网段的数据
    rule deny ip source 192.168.2.0 0.0.0.255 destination any
    # 拦截来至 192.168.3.0/24 网段的数据
    rule deny ip source 192.168.3.0 0.0.0.255 destination any
    quit
    display acl 3001

    # 创建并进入端口组一
    port-group 1
    # 添加端口到端口组
    group-member GigabitEthernet0/0/1 to GigabitEthernet0/0/3
    # 设置出口数据拦截为 acl 3001
    traffic-filter outbound acl 3001
    quit
    ```

    ### 相关资料

    * [eNSP 工程下载](file/demo1.zip)
    * [LSW1 交换机配置文件下载](file/demo1.cfg)
