# CentOS7 Samba

- 关闭 SeLinux

  ```bash
  setenforce 0
  sed -i 's/SELINUX *= *enforcing/SELINUX=disabled/g' /etc/selinux/config
  getenforce
  ```

- 防火墙放行

  ```bash
  firewall-cmd --add-service=samba
  firewall-cmd --add-service=samba --permanent
  ```

- 安装 samba

  ```bash
  yum install -y samba
  ```

- 创建系统用户

  ```bash
  groupadd test
  useradd -s /sbin/nologin -g test -M test
  ```

- 创建 samba 用户

  ```bash
  pdbedit -a -u test
  ```

- 创建存储目录

  ```bash
  mkdir -p -m 0777 /data/samba/test
  ```

- 修改配置文件

  修改`/etc/samba/smb.conf`文件

  ```properties
  [global]
          workgroup = SAMBA
          security = user

          passdb backend = tdbsam

          printing = cups
          printcap name = cups
          load printers = yes
          cups options = raw

  [test]
          comment = test
          path = /data/samba/test
          browseable = no
          create mode = 0777
          directory mode = 0777
          valid users = @test,test
          read list =test
          write list =test

  ```

- 启动服务

  ```bash
  systemctl restart smb.service
  systemctl status smb.service
  ```
