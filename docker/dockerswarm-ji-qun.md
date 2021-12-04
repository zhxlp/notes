# DockerSwarm 集群

## 创建集群

```properties
docker swarm init --advertise-addr <MANAGER-IP>
# --advertise-addr 指定与其他 node 通信的地址,非必填
```

Docker Swarm 集群间通信需要使用**2377/tcp 7946/tcp 7946/udp 4789/udp**端口

```properties
# 防火墙方向端口
firewall-cmd --add-service=docker-swarm
firewall-cmd --add-service=docker-swarm --permanent
```

## 获取 Join Token

```properties
# Manager Token
docker swarm join-token manager
# Worker Token
docker swarm join-token worker
```

## 添加节点

```properties
# 添加Manager节点,Manager节点数量最好为奇数,以防脑裂
docker swarm join-token manager
docker swarm join [OPTIONS] HOST:PORT
# 添加Worker节点
docker swarm join-token worker
docker swarm join [OPTIONS] HOST:PORT
```

## Service

```properties
# 创建Service
docker service create [OPTIONS] IMAGE [COMMAND] [ARG...]
# 查看Service 列表
docker service ls
# 查看Service 任务
docker service ps [OPTIONS] SERVICE [SERVICE...]
# 查看Service详细信息
docker service inspect [OPTIONS] SERVICE [SERVICE...]
# 扩展Service
docker service scale SERVICE=REPLICAS [SERVICE=REPLICAS...]
# 删除Service
docker service rm SERVICE [SERVICE...]
```
