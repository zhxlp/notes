# OpenStack VM 配置 Virtual IP

## 环境

- NetWork

  - 内部

    网段:172.16.1.0/24

    ID:aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa

  - 外部

    网段:192.168.1.0/24

    ID:bbbbbbbb-bbbb-bbbb-bbbb-bbbbbbbbbbbb

- node1

  IP:172.16.1.11

  Port ID:cccccccc-cccc-cccc-cccc-cccccccccccc

- node2

  IP:172.16.1.12

  Port ID:dddddddd-dddd-dddd-dddd-dddddddddddd

- Virtual IP

  IP:172.16.0.10
  Port ID:eeeeeeee-eeee-eeee-eeee-eeeeeeeeeeee

## 配置

- 查看网络列表

  ```bash
  openstack network list
  ```

- 创建网络端口

  ```bash
  # 在内部网络创建Virtual IP端口
  openstack port create --network aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa vip
  ```

- 添加允许 IP

  ```bash
  # 172.16.1.11端口添加允许IP 172.16.1.10
  openstack port set --allowed-address ip-address=172.16.1.10 cccccccc-cccc-cccc-cccc-cccccccccccc
  # 172.16.1.12端口添加允许IP 172.16.1.10
  openstack port set --allowed-address ip-address=172.16.1.10 dddddddd-dddd-dddd-dddd-dddddddddddd
  ```

- 创建浮动 IP

  ```bash
  # 在外部网络创建浮动IP
  openstack floating ip create bbbbbbbb-bbbb-bbbb-bbbb-bbbbbbbbbbbb
  ```

- 查看浮动 IP 列表

  ```bash
  openstack floating ip list
  ```

- 浮动 IP 绑定端口

  ```bash
  # 192.168.1.10浮动IP绑定 172.16.1.10端口
  openstack floating ip set --port eeeeeeee-eeee-eeee-eeee-eeeeeeeeeeee 192.168.1.10
  ```

```


```
