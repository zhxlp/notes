# Firewalld Rich Language

参考连接：https://access.redhat.com/documentation/zh-cn/red_hat_enterprise_linux/7/html/security_guide/sec-using_firewalls

## 格式

- 增加一条规则命令

  ```bash
  firewall-cmd [--zone=zone] --add-rich-rule='rule' [--timeout 9=seconds]
  ```

- 移除一条规则命令

  ```bash
  firewall-cmd [--zone=zone] --remove-rich-rule='rule'
  ```

- 检查规则是否存在

  ```bash
  firewall-cmd [--zone=zone] --query-rich-rule='rule'
  ```

- 多规则命令的结构

  ```bash
  rule [family="<rule family>"]
      [ source address="<address>" [invert="True"] ]
      [ destination address="<address>" [invert="True"] ]
      [ <element> ]
      [ log [prefix="<prefix text>"] [level="<log level>"] [limit value="rate/duration"] ]
      [ audit ]
      [ accept|reject|drop ]
  ```

  **family**：规则系列, `IPv4` 或 `IPv6`

  **source address**：源地址，一个源地址或者地址范围是一个为 `IPv4` 或者 `IPv6` 做掩护的 IP 地址或者一个网络 IP 地址，可以通过增加 `invert`="_true_"来颠倒源地址命令的意思

  **destination address**：目的地址，目标地址使用跟源地址相同的语法

  **element**：基本要素，这个要素只可以是以下要素类型之一： `service` ， `port` ， `protocol` ， `masquerade` ， `icmp-block` 和 `forward-port` 。

  ```properties
  # service
  # 服务名称是 firewalld 提供的其中一种服务。要获得被支持的服务的列表，输入以下命令:firewall-cmd --get-services
  service name=service_name
  # port
  # 端口既可以是一个独立端口数字，又或者端口范围，例如，5060-5062。协议可以指定为 tcp 或 udp
  port port=number_or_range protocol=protocol
  # protocol
  # 协议值可以是一个协议 ID 数字，或者一个协议名。预知可用协议，请查阅 /etc/protocols
  protocol value=protocol_name_or_ID
  # icmp-block
  # 用这个命令阻绝一个或多个 ICMP 类型。 ICMP 类型是 firewalld 支持的 ICMP 类型之一。要获得被支持的 ICMP 类型列表,输入一下命令：firewall-cmd --get-icmptypes
  icmp-block name=icmptype_name
  # masquerade
  # 打开规则里的 IP 伪装。用源地址而不是目的地址来把伪装限制在这个区域内。在此，指定一个动作是不被允许的
  # forward-port
  # 从一个带有指定为 tcp 或 udp 协议的本地端口转发数据包到另一个本地端口，或另一台机器，或另一台机器上的另一个端口。 port 和 to-port 可以是一个单独的端口数字，或一个端口范围。而目的地址是一个简单的 IP 地址。在此，指定一个动作是不被允许的。 forward-port 命令使用内部动作 accept
  forward-port port=number_or_range protocol=protocol /
              to-port=number_or_range to-addr=address
  ```

  **log**：注册含有内核记录的新的连接请求到规则中，比如系统记录。您可以定义一个前缀文本——可以把记录信息作为前缀加入。记录等级可以是 `emerg` 、 `alert` 、 `crit` 、 `error` 、`warning` 、 `notice` 、 `info` 或者 `debug` 中的一个。可以选择记录的用法，可以按以下方式限制注册

  ```properties
  log [prefix=prefix text] [level=log level] limit value=rate/duration
  # 等级用正的自然数 [1, ..] 表达，持续时间的单位为 s 、 m 、 h 、 d 。 s 表示秒， m 表示分钟， h 表示小时， d 表示天。最大限定值是 1/d ，意为每天最多有一条日志进入
  ```

  **audit**：审核为发送到 `auditd` 服务的审核记录来注册提供了另一种方法。审核类型可以是 `ACCEPT` 、 `REJECT` 或 `DROP` 中的一种，但不能在 `audit` 命令后指定，因为审核类型将会从规则动作中自动收集。审核不包含自身参数，但可以选择性地增加限制。审核的使用是可选择的

  **accept|reject|drop**：可以是 `accept` 、`reject` 或 `drop` 中的一个行为。规则中仅仅包含一个要素或者来源。如果规则中包含一个要素，那么行为可以处理符合要素的新连接。如果规则中包含一个来源，那么指定的行为可以处理来自源地址的一切内容

  ```properties
  accept | reject [type=reject type] | drop
  ```

## 示例

- 限制 ssh 登录 IP 为`192.168.2.200`

  ```bash
  firewall-cmd --remove-service=ssh
  firewall-cmd --add-rich-rule='rule family="ipv4" source address="192.168.2.200" service name="ssh" accept'
  ```

- 2222 端口转发到`192.168.2.39`的 22 端口

  ```bash
  firewall-cmd --add-rich-rule='rule family="ipv4" masquerade'
  firewall-cmd --add-rich-rule='rule family="ipv4" forward-port port="2222" protocol="tcp" to-port="22" to-addr="192.168.2.39"'
  ```

- 放行 3389 端口 1 小时

  ```bash
  firewall-cmd --add-rich-rule='rule port port="3389" protocol="tcp" accept' --timeout=1h
  ```

- 拦截 icmp 请求

  ```bash
  firewall-cmd --add-rich-rule='rule protocol value="icmp" drop'
  ```
