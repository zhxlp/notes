# CentOS7 安装 OpenVPN

## 服务端

- 环境

  操作系统：CentOS 7.6.1810

  OpenVPN：2.4.9

  easy-ras：3.0.7

- 安装`epel`源

  ```bash
  yum install -y epel-release
  ```

- 安装软件包

  ```bash
  yum install -y openvpn easy-rsa
  ```

- 开启转发功能

  ```bash
  echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.conf
  sysctl -p
  ```

- 拷贝配置文件

  ```bash
  cp /usr/share/doc/openvpn-2.4.9/sample/sample-config-files/server.conf /etc/openvpn
  ```

- 修改`/etc/openvpn/server.conf`文件

  ```properties
  # 使用1194端口
  port 1194
  # 使用tcp协议
  proto tcp
  # 使用tun虚拟网卡设备（还有一种是Tap）
  dev tun
  # 指定server端证书路径
  ca ca.crt
  # 指定server端证书路径
  cert server.crt
  # 指定server端密钥路径
  key server.key
  # 定义Diffie hellman文件
  dh dh2048.pem
  # 为了获得SSL/TLS提供的额外安全性，创建一个“HMAC防火墙”来帮助阻止DoS攻击和UDP端口泛滥
  tls-auth ta.key 0
  cipher AES-256-CBC
  # 启用vpn子网
  topology subnet
  # openvpn 网段
  server 10.8.0.0 255.255.255.0
  # 添加openvpn路由
  push "route 10.8.0.0 255.255.0.0"
  # 向客户端推送的DNS信息
  push "dhcp-option DNS 223.5.5.5"
  # 记录虚拟IP地址与客户端的关联记录
  ifconfig-pool-persist ipp.txt
  # 保持VPN会话,每10秒Ping一次,如果在120秒的时间段内没有收到Ping,则假设远程对等机关闭
  keepalive 10 120
  # 开启Lzo数据压缩
  comp-lzo
  # 运行openvpn的用户和组
  user nobody
  group nobody
  # 通过keepalive检测超时后,重新启动VPN,不重新读取keys,保留第一次使用的keys
  persist-key
  # 通过keepalive检测超时后,重新启动VPN,一直保持tun或者tap设备是linkup的.否则网络连接,会先linkdown然后再linkup
  persist-tun
  # 当前连接信息,每分钟重写一次
  status openvpn-status.log
  # 日志记录冗长级别
  verb 3
  # 配置客户端静态IP
  client-config-dir ccd
  ```

- 创建`ccd`目录

  ```bash
  mkdir /etc/openvpn/ccd
  ```

- 复制`easy-rsa`脚本

  ```bash
  mkdir -p /etc/openvpn/easy-rsa/
  cp -rf /usr/share/easy-rsa/3.0.7/* /etc/openvpn/easy-rsa
  ```

- 切换`/etc/openvpn/easy-rsa`目录

  ```bash
  cd /etc/openvpn/easy-rsa
  ```

- 创建服务的证书

  ```bash
  #建立一个空的pki结构，生成一系列的文件和目录
  ./easyrsa init-pki
  #创建ca  密码 和 cn那么需要记住
  ./easyrsa build-ca nopass
  #创建服务端证书  common name 最好不要跟前面的cn那么一样
  ./easyrsa gen-req server nopass
  #签约服务端证书
  ./easyrsa sign server server
  #创建Diffie-Hellman
  ./easyrsa gen-dh
  # 创建ta.key
  openvpn --genkey --secret /etc/openvpn/ta.key
  ```

- 拷贝证书到`openvpn`配置目录

  ```bash
  cp /etc/openvpn/easy-rsa/pki/dh.pem /etc/openvpn/dh2048.pem
  cp /etc/openvpn/easy-rsa/pki/ca.crt /etc/openvpn/
  cp /etc/openvpn/easy-rsa/pki/issued/server.crt /etc/openvpn/
  cp /etc/openvpn/easy-rsa/pki/private/server.key /etc/openvpn/
  ```

- 启动服务

  ```bash
  systemctl start openvpn@server.service
  systemctl status openvpn@server.service
  systemctl enable openvpn@server.service
  ```

- 配置防火墙

  ```bash
  firewall-cmd --add-port=1194/tcp
  firewall-cmd --permanent --add-port=1194/tcp
  # firewall-cmd --permanent --add-masquerade
  # firewall-cmd --permanent --direct --passthrough ipv4 -t nat -A POSTROUTING -s 10.8.0.0/24 -o ens33 -j MASQUERADE
  # firewall-cmd --reload
  ```

## 客户端配置

- 登录服务端，创建客户端证书

  ```bash
  cd /etc/openvpn/easy-rsa
  # 创建 zhxlp的证书，zhxlp客户端名称，唯一
  ./easyrsa gen-req zhxlp nopass
  # 签约zhxlp证书
  ./easyrsa sign client zhxlp
  ```

- 下载证书拷贝到客户端`~`目录

  ```properties
  /etc/openvpn/easy-rsa/pki/ca.crt
  /etc/openvpn/easy-rsa/pki/private/zhxlp.key
  /etc/openvpn/easy-rsa/pki/issued/zhxlp.crt
  /etc/openvpn/ta.key
  ```

- 配置客户端固定 IP（可选）

  ```bash
  echo 'ifconfig-push 10.8.0.10 255.255.255.0' > /etc/openvpn/ccd/zhxlp
  ```

- 安装软件包

  ```bash
  yum install -y epel-release
  yum install -y openvpn
  ```

- 拷贝证书到配置目录

  ```bash
  cp ~/ca.crt /etc/openvpn/
  cp ~/zhxlp.key /etc/openvpn/
  cp ~/zhxlp.crt /etc/openvpn/
  cp ~/ta.key /etc/openvpn/
  ```

- 配置客户端配置文件

  ```bash
  cat > /etc/openvpn/client.conf << EOF
  # 使用TLS加密传输
  tls-client
  # 配置证书和密钥路径
  ca ca.crt
  cert zhxlp.crt
  key zhxlp.key
  # 指定协议
  proto tcp
  # 指定服务器地址和端口
  remote 192.168.2.55 1194 tcp-client
  # 虚拟网卡类型
  dev tun
  # vpn子网
  topology subnet
  # 为了获得SSL/TLS提供的额外安全性
  tls-auth ta.key 1
  cipher AES-256-CBC
  # 数据压缩
  comp-lzo
  # 连接状态
  status openvpn-status.log
  pull
  EOF
  ```

- 启动服务

  ```bash
  systemctl start openvpn@client.service
  systemctl status openvpn@client.service
  systemctl enable openvpn@client.service
  ```
