# Webmin 安装

## 安装配置

- 下载软件包

  ```bash
  wget https://prdownloads.sourceforge.net/webadmin/webmin-1.960-1.noarch.rpm
  ```

- 安装

  ```bash
  yum install -y ./webmin-1.960-1.noarch.rpm
  ```

- 防火墙放行

  ```bash
  firewall-cmd --add-port=10000/tcp --permanent
  firewall-cmd --add-port=10000/tcp
  ```

- 访问

  浏览器打开 `https://you_ip:10000`

## 常见问题

### 创建 samba 目录无法使用中文名

修改`/usr/libexec/webmin/samba/save_fshare.cgi`文件 60 行左右

```properties
#	elsif ($name !~ /^[A-Za-z0-9_\$\-\. ]+$/) {
#		&error(&text('savefshare_mode', $name));
#	}
```

### 保存后 samba 配置文件乱码

修改``/usr/libexec/webmin/samba/samba-lib.pl`文件 122 行和 133 行左右

```properties
				#&print_tempfile(CONF, "\t$k = ",
				#	&to_utf8($share{$k}),"\n");
				&print_tempfile(CONF, "\t$k = $share{$k}\n");
```
