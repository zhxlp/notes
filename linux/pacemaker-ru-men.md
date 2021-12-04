# Pacemaker 入门

参照[官方文档](http://clusterlabs.org/quickstart-redhat.html)

所有示例都假设两个节点可通过其短名称和 IP 地址访问：

- `node1 - 192.168.1.1`
- `node2 - 192.168.1.2`

遵循的惯例是**[ALL]＃** 表示需要在所有集群计算机上运行的命令，**[ONE]＃**表示只需要在一个集群主机上运行的命令。

# RHEL 7

## 安装

Pacemaker 作为红帽 [高可用性附加组件的一部分提供](https://www.redhat.com/en/resources/high-availability-add-datasheet)。在 RHEL 上试用它的最简单方法是从[Scientific Linux](https://scientificlinux.org/) 或[CentOS](https://www.centos.org/)存储库安装它 。

如果您已经在运行 CentOS 或 Scientific Linux，则可以跳过此步骤。否则，要教机器在哪里找到 CentOS 包，运行：

```bash
[ALL] # cat < /etc/yum.repos.d/centos.repo
[centos-7-base]
name=CentOS-$releasever - Base
mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=os
#baseurl=http://mirror.centos.org/centos/$releasever/os/$basearch/
enabled=1
EOF
```

接下来我们使用 yum 来安装起搏器和我们需要的一些其他必要的包：

```bash
[ALL] # yum install -y pacemaker pcs resource-agents
```

防火墙放行服务端口

```bash
[ALL] # firewall-cmd --permanent --add-service=high-availability
[ALL] # firewall-cmd --reload
```

## 创建群集

RHEL7 上支持的堆栈基于 Corosync 2，因此 Pacemaker 也使用它。

首先确保**pcs 守护程序**在每个节点上运行：

```bash
[ALL] # systemctl start pcsd.service
[ALL] # systemctl enable pcsd.service
```

然后我们设置**pcs**所需的身份验证。

```bash
[ALL] # echo hacluster | passwd --stdin hacluster
[ONE] # pcs cluster auth node1 node2 -u hacluster -p hacluster --force
```

我们现在创建一个集群并用一些节点填充它。请注意，名称不能超过 15 个字符（我们将使用'pacemaker1'）。

```bash
[ONE] # pcs cluster setup --force --name pacemaker1 node1 node2
```

## 启动群集

```bash
[ONE] # pcs cluster start --all
```

## 设置群集选项

有这么多设备和可能的拓扑结构，几乎不可能在这样的文档中包含 Fencing。现在我们将禁用它。

```bash
[ONE] # pcs property set stonith-enabled=false
```

部署 Pacemaker 的最常见方法之一是采用双节点配置。但是，在这种情况下，仲裁作为一个概念是没有意义的（因为只有超过一半的节点可用时才有它），所以我们也会禁用它。

```bash
[ONE] # pcs property set no-quorum-policy=ignore
```

出于演示目的，我们将强制群集在单个故障后移动服务：

```bash
[ONE] # pcs resource defaults migration-threshold=1
```

## 添加资源

让我们添加一个集群服务，我们将选择一个不需要任何配置，并在任何地方工作，使事情变得简单。这是命令：

```bash
[ONE] # pcs resource create my_first_svc Dummy op monitor interval=120s
```

“ **my_first_svc** ”是服务的名称。

“ **ocf：pacemaker：Dummy** ”告诉 Pacemaker 使用哪个脚本（[Dummy](https://github.com/ClusterLabs/pacemaker/blob/master/extra/resources/Dummy) - 一个有用的模板和类似指南的代理），它所在的命名空间（起搏器）以及它符合的标准（[OCF](http://clusterlabs.org/pacemaker/doc/en-US/Pacemaker/1.1/html/Pacemaker_Explained/s-resource-supported.html#_open_cluster_framework)）。

“ **op monitor interval = 120s** ”告诉 Pacemaker 通过调用代理的**监视器**操作每 2 分钟检查一次该服务的运行状况。

您现在应该能够看到正在运行的服务：

```bash
[ONE] # pcs status
```

要么

```bash
[ONE] # crm_mon -1
```

## 模拟服务失败

我们可以通过告诉服务直接停止（不告诉集群）来模拟错误：

```bash
[ONE] # crm_resource --resource my_first_svc --force-stop
```

如果现在以交互模式（默认值）运行**crm_mon**，您应该看到（在 2 分钟的监视间隔内）群集通知**my_first_svc**失败并将其移动到另一个节点。
