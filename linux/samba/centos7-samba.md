# CentOS7安装Samba

*   关闭 SeLinux

    ```bash
    setenforce 0
    sed -i 's/SELINUX *= *enforcing/SELINUX=disabled/g' /etc/selinux/config
    getenforce
    ```
*   防火墙放行

    ```bash
    firewall-cmd --add-service=samba
    firewall-cmd --add-service=samba --permanent
    ```
*   安装 samba

    ```bash
    yum install -y samba
    ```
*   启动服务并设置开机自启

    ```bash
    systemctl restart smb.service
    systemctl enable smb.service
    systemctl status smb.service
    ```





