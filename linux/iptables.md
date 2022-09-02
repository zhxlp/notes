# iptables

## iptables

参考链接：http://www.zsythink.net/archives/1199

### 格式

*   命令格式

    ```properties
    # 追加/检查是否存在/删除 规则
    iptables [-t table] {-A|-C|-D} chain rule-specification
    ip6tables [-t table] {-A|-C|-D} chain rule-specification
    # 插入规则，可以指定规则位置
    iptables [-t table] -I chain [rulenum] rule-specification
    # 替换指定规则
    iptables [-t table] -R chain rulenum rule-specification
    # 根据位置删除规则
    iptables [-t table] -D chain rulenum
    # 打印规则保存为配置文件时的内容
    iptables [-t table] -S [chain [rulenum]]

    iptables [-t table] {-F|-L|-Z} [chain [rulenum]] [options...]

    iptables [-t table] -N chain

    iptables [-t table] -X [chain]

    iptables [-t table] -P chain target

    iptables [-t table] -E old-chain-name new-chain-name

    rule-specification = [matches...] [target]

    match = -m matchname [per-match-options]

    target = -j targetname [per-target-options]
    ```
*   table

    ```properties
    filter
    nat
    mangle
    raw
    ```
*   chain

    ```properties
    PREROUTING
    INPUT
    OUTPUT
    FORWARD
    POSTROUTING
    ```
*   target

    ```properties
    # 允许数据包通过
    ACCEPT
    # 直接丢弃数据包，不给任何回应信息
    DROP
    # 拒绝数据包通过，必要时会给数据发送端一个响应的信息
    REJECT
    # 源地址转换，解决内网用户用同一个公网地址上网的问题
    SNAT
    # 是SNAT的一种特殊形式，适用于动态的、临时会变的ip上
    MASQUERADE
    # 目标地址转换
    DNAT
    # 在本机做端口映射
    REDIRECT
    # 在/var/log/messages文件中记录日志信息，然后将数据包传递给下一条规则，也就是说除了记录以外不对数据包做任何其他操作，仍然让下一条规则去匹配
    LOG
    ```
*   数据经过防火墙的流程

    <img src="../.gitbook/assets/iptables-1.png" alt="iptables详解（1）：iptables概念" data-size="original">

    ### 示例
*   默认配置

    ```bash
    # 设置默认策略
    iptables -P INPUT ACCEPT
    iptables -P FORWARD ACCEPT
    iptables -P OUTPUT ACCEPT
    # 清空规则
    iptables -F
    # 允许进入的数据包只能是刚刚我发出去的数据包的回应，ESTABLISHED：已建立的链接状态。RELATED：该数据包与本机发出的数据包有关
    iptables -A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
    # -i参数是指定接口，这里的接口是lo ，lo就是Loopback（本地环回接口），意思就允许本地环回接口在INPUT表的所有数据通信
    iptables -A INPUT -i lo -j ACCEPT
    # 该规则的意思就是新连接ssh默认端口22的允许进入
    iptables -A INPUT -p tcp -m state --state NEW -m tcp --dport 22 -j ACCEPT
    # INPUT表中拒绝所有其他不符合上述任何一条规则的数据包。并且发送一条host prohibited的消息给被拒绝的主机
    iptables -A INPUT -j REJECT --reject-with icmp-host-prohibited
    # FORWARD表中拒绝所有其他不符合上述任何一条规则的数据包。并且发送一条host prohibited的消息给被拒绝的主机
    iptables -A FORWARD -j REJECT --reject-with icmp-host-prohibited
    ```
*   永久保存规则

    ```bash
    iptables-save > /etc/sysconfig/iptables
    ```
*   查看表规则

    ```bash
    # 默认查看 filter表
    # -n 端口和地址以数字显示
    ```

## -v verbose mode(详细模式)

iptables -nvL iptables -t filter -nvL

## 查看 filter 表 INPUT 链记录

iptables -nvL INPUT

## --line 显示行号

iptables --line -nL

````
* 清空表规则

```bash
iptables -F
iptables -t filter -F
iptables -F INPUT
````

*   放行 22 端口

    ```bash
    iptables -t filter -I INPUT -p tcp --dport 22 -j ACCEPT
    ```
*   放行`192.168.2.200`

    ```bash
    iptables -t filter -I INPUT -s 192.168.2.200 -j ACCEPT
    ```
*   禁止 icmp

    ```bash
    iptables -t filter -I INPUT -p icmp -j DROP
    ```
