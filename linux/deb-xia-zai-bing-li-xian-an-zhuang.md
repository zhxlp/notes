# deb下载并离线安装

在系统mini环境或docker环境执行下面命令获取软件及依赖deb包地址

```
apt-get install --print-uris wget | grep -oP "http.*?deb"
```

通过wget 获取迅雷下载



安装

```
apt-get install ./*.deb
```

