# HyperLedger-Fabric 基础环境

## 环境

- 操作系统：CentOS7.6
- HyperLedger-Fabric:1.4.2
- Docker: > 17.06.2
- Docker-Compose: > 1.14.0
- Golang: > 1.11.5
- NodeJs: > 10.15.3
- npm: > 5.5.1

## 安装 docker 和 docker-compose

- 安装软件包

  ```bash
  # 安装软件包
  yum install -y yum-utils
  yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
  yum install -y http://mirrors.aliyun.com/epel/epel-release-latest-7.noarch.rpm
  yum install -y docker-ce docker-compose
  # 查看软件版本
  docker --version  # >17.06.2
  docker-compose --version  # >1.14.0
  ```

- 配置 docker 镜像加速

  ```bash
  mkdir -p /etc/docker
  echo '
  {
   "registry-mirrors": ["https://9cpn8tt6.mirror.aliyuncs.com"]
  }
  ' > /etc/docker/daemon.json
  systemctl restart docker.service
  systemctl enable docker.service

  ```

## 安装 NodeJs

- 安装软件包

  ```bash
  # 下载二进制文件
  wget https://nodejs.org/dist/v10.16.3/node-v10.16.3-linux-x64.tar.gz
  # 解压软包
  tar -xvf node-v10.16.3-linux-x64.tar.gz -C /opt/
  mv /opt/node-v10.16.3-linux-x64/ /opt/nodejs
  # 配置环境变量
  echo '
  # nodejs
  export PATH=$PATH:/opt/nodejs/bin/
  ' >> /etc/profile
  source /etc/profile
  # 查看软件版本
  node -v # > 10.15.3
  npm -version # > 5.5.1
  ```

- 配置 npm taobao 源

  ```bash
  npm install -g nrm --registry=https://registry.npm.taobao.org
  nrm use taobao
  ```

## 安装 Golang

- 安装软件包

  ```bash
  # 下载二进制文件
  wget https://studygolang.com/dl/golang/go1.11.5.linux-amd64.tar.gz
  # 解压软件包
  tar -xvf go1.11.5.linux-amd64.tar.gz -C /opt/
  # 配置环境变量
  echo '
  # go
  export GOROOT=/opt/go
  export PATH=$PATH:$GOROOT/bin
  export GOPATH=/opt/gopath
  export PATH=$PATH:$GOPATH/bin
  ' >> /etc/profile
  source /etc/profile
  # 查看软件版本
  go version # > 1.11.x

  ```

## 安装 Fabric 相关文件

有两种方式安装，编译安装和下载二进程文件安装，任选其它

- 下载源码

  ```bash
  mkdir -p $GOPATH/src/github.com/hyperledger/
  cd $GOPATH/src/github.com/hyperledger/
  yum install -y git gcc
  git clone -b v1.4.2 https://github.com/hyperledger/fabric.git
  git clone -b v1.4.2 https://github.com/hyperledger/fabric-samples.git
  git clone -b v1.4.2 https://github.com/hyperledger/fabric-ca.git
  ```

- 编译安装

  ```bash
  # 编译fabric
  cd $GOPATH/src/github.com/hyperledger/fabric
  make release
  # 编译fabric-ca
  cd $GOPATH/src/github.com/hyperledger/fabric-ca
  make fabric-ca-server
  make fabric-ca-client
  # 拷贝二进程文件
  cp -rf $GOPATH/src/github.com/hyperledger/fabric/release/linux-amd64/bin $GOPATH/
  cp -rf $GOPATH/src/github.com/hyperledger/fabric-ca/bin $GOPATH/
  # 加载环境变量
  source /etc/profile
  ```

- 下载二进程文件安装

  ```bash
  wget https://nexus.hyperledger.org/content/repositories/releases/org/hyperledger/fabric/hyperledger-fabric/linux-amd64-1.4.2/hyperledger-fabric-linux-amd64-1.4.2.tar.gz
  tar xvf hyperledger-fabric-linux-amd64-1.4.2.tar.gz -C $GOPATH/

  wget https://nexus.hyperledger.org/content/repositories/releases/org/hyperledger/fabric-ca/hyperledger-fabric-ca/linux-amd64-1.4.2/hyperledger-fabric-ca-linux-amd64-1.4.2.tar.gz
  tar xvf hyperledger-fabric-ca-linux-amd64-1.4.2.tar.gz -C $GOPATH/

  source /etc/profile

  ```

## 下载相关 docker 镜像

```bash
docker pull hyperledger/fabric-peer:1.4.2
docker pull hyperledger/fabric-orderer:1.4.2
docker pull hyperledger/fabric-ccenv:1.4.2
docker pull hyperledger/fabric-javaenv:1.4.2
docker pull hyperledger/fabric-tools:1.4.2
docker pull hyperledger/fabric-ca:1.4.2
docker pull hyperledger/fabric-couchdb:0.4.15
docker pull hyperledger/fabric-kafka:0.4.15
docker pull hyperledger/fabric-zookeeper:0.4.15
```
