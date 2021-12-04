# Dockerfile

### FROM

```properties
FROM [--platform=<platform>] <image>[:<tag>] [AS <name>]
# platform 指定镜像的系统架构，如linux/amd64、linux/arm64、windows/amd64，默认与构建镜像的平台一致
```

FROM 指令设置基础镜像，镜像在基础镜像上进行构建

```properties
# 在centos7.6.1810镜像基础上构建新镜像
FROM centos:7.6.1810
```

### RUN

```properties
RUN <command>
RUN ["executable", "param1", "param2"]
```

RUN 命令执行命令并创建新的镜像层，通常用于安装软件包

```properties
# 打印hello
RUN echo hello
RUN ["/bin/bash", "-c", "echo hello"]
# 安装httpd并清除缓存
RUN set -eux; \
    yum -y update ; \
    yum install -y httpd ; \
    yum clean all
```

### CMD

```properties
CMD ["executable","param1","param2"]
CMD ["param1","param2"] (作为ENTRYPOINT的默认参数)
CMD command param1 param2
```

CMD 命令设置容器启动后默认执行的命令及其参数，但 CMD 设置的命令能够被`docker run`命令后面的命令行参数替换

每个 Dockerfile 中只能有一个 CMD，当指定多个时，只有最后一个起效

> 注意：
>
> Docker 容器启动时，默认会把容器内部第一个进程，也就是`pid=1`的程序，作为 docker 容器是否正在运行的依据，如果 docker 容器 pid=1 的进程挂了，那么 docker 容器便会直接退出。

```properties
# 启动nginx
CMD ["nginx", "-g", "daemon off;"]
```

### ENTRYPOINT

```properties
ENTRYPOINT ["executable", "param1", "param2"]
ENTRYPOINT command param1 param2
```

ENTRYPOINT 配置容器启动时的执行命令（不会被忽略，一定会被执行，即使运行 `docker run`时指定了其他命令）

每个 Dockerfile 中只能有一个 ENTRYPOINT，当指定多个时，只有最后一个起效

```properties
# redis 镜像的内容
COPY docker-entrypoint.sh /usr/local/bin/
ENTRYPOINT ["docker-entrypoint.sh"]

EXPOSE 6379
CMD ["redis-server"]
```

### LABEL

```properties
LABEL <key>=<value> <key>=<value> <key>=<value> ...
```

LABEL 指令为镜像添加元数据

```properties
# 添加作者信息
LABEL maintainer="NGINX Docker Maintainers <docker-maint@nginx.com>"
```

### EXPOSE

```properties
EXPOSE <port> [<port>/<protocol>...]
# 默认tcp协议
```

EXPOSE 指令暴露容器的端口

```properties
# nginx 暴露80端口
EXPOSE 80
```

### ENV

```properties
ENV <key> <value>
ENV <key>=<value> ...
```

ENV 指令设置镜像构建过程中的环境变量

```properties
ENV MYDIR /mydir
RUN mkdir $MYDIR
```

### WORKDIR

```properties
WORKDIR path
```

WORKDIR 指令设置工作目录

```properties
WORKDIR /workpath
```

### ADD

```properties
ADD [--chown=<user>:<group>] <src>... <dest>
ADD [--chown=<user>:<group>] ["<src>",... "<dest>"]
```

ADD 指令拷贝本地资源或 url 资源到镜像，复制目录时，目录本身不会被复制，只是其内容被复制。本地资源为压缩包时，会解压文件，url 资源不会

```properties
ADD test /test
```

### COPY

```properties
COPY [--chown=<user>:<group>] <src>... <dest>
COPY [--chown=<user>:<group>] ["<src>",... "<dest>"]
```

COPY 指令拷贝本地资源到镜像，复制目录时，目录本身不会被复制，只是其内容被复制。

```properties
COPY test /test
```

### VOLUME

```properties
VOLUME path
VOLUME ["/path"]
```

VOLUME 指令将宿主机本地目录挂载到镜像的指定目录，默认是本地`/var/lib/docker/volumes/***`

```properties
# mysql
VOLUME [/var/lib/mysql]
```

### USER

```properties
USER <user>[:<group>]
USER <UID>[:<GID>]
```

USER 指令指定后续命令使用此用户进行，如 RUN、CMD 等。镜像构建完成后，通过 docker run 运行容器时，可以通过-u 参数来覆盖所指定的用户

```properties
USER www
```

### ARG

```properties
ARG <name>[=<default value>]
```

ARG 指令定义一个变量，用户可以在`docker build`时，添加`--build-arg <varname>=<value>` 参数将变量传递给构建器

使用`ENV`指令定义的环境变量 始终会覆盖`ARG`指令的同名变量

```properties
FROM busybox
ARG SETTINGS
RUN ./run/setup $SETTINGS
```

### ONBUILD

```properties
ONBUILD [INSTRUCTION]
```

`ONBUILD` 是一个特殊的指令，它后面跟的是其它指令，比如 `RUN`, `COPY` 等，而这些指令，在当前镜像构建时并不会被执行。只有当以当前镜像为基础镜像，去构建下一级镜像的时候才会被执行

```properties
FROM node:slim
RUN mkdir /app
WORKDIR /app
ONBUILD COPY ./package.json /app
ONBUILD RUN [ "npm", "install" ]
ONBUILD COPY . /app/
CMD [ "npm", "start" ]
```

### STOPSIGNAL

```properties
STOPSIGNAL signal
```

### HEALTHCHECK

```properties
HEALTHCHECK [OPTIONS] CMD command
HEALTHCHECK NONE
```

### SHELL

```properties
SHELL ["executable", "parameters"]
```
