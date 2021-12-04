# CentOS7 安装 Docker

- 安装依赖

  ```bash
  yum install -y yum-utils device-mapper-persistent-data lvm2
  ```

- 添加 docker-ce 源

  ```bash
  yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
  ```

- 安装 docker-ce

  ```bash
  yum install -y docker-ce docker-ce-cli containerd.io
  ```

- 配置 docker 加速

  ```bash
  mkdir -p /etc/docker
  tee /etc/docker/daemon.json << EOF
  {
    "registry-mirrors": [
      "https://registry.docker-cn.com",
      "https://docker.mirrors.ustc.edu.cn",
      "http://hub-mirror.c.163.com"
    ]
  }
  EOF
  ```

- 启动 docker 并设置开机自启

  ```bash
  systemctl restart docker.service
  systemctl enable docker.service
  ```
