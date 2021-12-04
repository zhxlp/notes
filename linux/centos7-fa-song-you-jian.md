# CentOS7 发送邮件

- 安装`mailx`。

  ```bash
  yum install -y mailx
  ```

- 配置邮箱信息

  编辑`/etc/mail.rc`添加一下内容

  ```properties
  set from=****@qq.com
  set smtp=smtps://smtp.qq.com:465
  set smtp-auth-user=****@qq.com
  set smtp-auth-password=*******
  set smtp-auth=login
  set nss-config-dir=/etc/pki/nssdb
  set smtp-user-starttls
  set ssl-verify=ignore
  ```

- 添加邮箱 ssl 证书到本地。

  ```bash
  # 获取证书内容
  echo -n | openssl s_client -connect smtp.qq.com:465 | sed -ne '/-BEGIN CERTIFICATE-/,/-END CERTIFICATE-/p' > /etc/pki/nssdb/qq.crt
  # 添加证书到数据库
  certutil -A -n "GeoTrust SSL CA" -t "C,," -d /etc/pki/nssdb -i /etc/pki/nssdb/qq.crt
  certutil -A -n "GeoTrust Global CA" -t "C,," -d /etc/pki/nssdb -i /etc/pki/nssdb/qq.crt
  # 列出指定目录下的证书
  certutil -L -d /etc/pki/nssdb
  # 指明受新人证书、防报错
  certutil -A -n "GeoTrust SSL CA - G3" -t "Pu,Pu,Pu" -d /etc/pki/nssdb -i /etc/pki/nssdb/qq.crt
  ```

- 测试

  ```bash
  # echo '邮件正文' | mailx -s '邮寄标题' '收件邮箱'
  echo 'Test Mail!!!' | mailx -s 'Test' '****@qq.com'
  ```
