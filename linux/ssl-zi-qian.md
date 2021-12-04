# SSL 自签

## 自签

- 生成 CA 私钥

  ```bash
  openssl genrsa -out ca.key 2048
  ```

- 生成 CA 证书

  ```bash
  openssl req -sha256 -new -x509 -days 7300 -key ca.key -out ca.crt \
      -subj "/C=CN/ST=Guangdong/L=Shenzhen/O=Zhxlp/OU=IT/CN=Zhxlp Root CA"
  ```

- 查看 CA 证书信息

  ```bash
  openssl x509 -in ca.crt -text -noout
  ```

- 生成 Server 私钥

  ```bash
  openssl genrsa -out server.key 2048
  ```

- 生成 Server 证书签名请求(CSR)

  ```bash
  openssl req -new -sha256 -key server.key \
      -subj "/C=CN/ST=Guangdong/L=Shenzhen/O=Zhxlp/OU=IT/CN=www.zhxlp.com" \
      -reqexts SAN \
      -config <(cat /etc/pki/tls/openssl.cnf \
          <(printf "[SAN]\nsubjectAltName=DNS:www.zhxlp.com,DNS:zhxlp.com,DNS:localhost,IP:127.0.0.1")) \
      -out server.csr
  ```

- 查看 Server 证书签名请求(CSR)

  ```bash
  openssl req -in server.csr -noout -text
  ```

- 生成 Server 证书

  ```bash
  openssl x509 -req -days 365 \
      -CAserial server.srl -CAcreateserial \
      -in server.csr -CA ca.crt -CAkey ca.key  \
      -extensions SAN \
      -extfile <(cat /etc/pki/tls/openssl.cnf \
          <(printf "[SAN]\nsubjectAltName=DNS:www.zhxlp.com,DNS:zhxlp.com,DNS:localhost,IP:127.0.0.1")) \
      -out server.crt
  ```

- 查看 Server 证书信息

  ```bash
  openssl x509 -in server.crt -noout -text
  ```

- 合并证书

  ```bash
  cat server.crt ca.crt > fullchain.crt
  ```

  ## 安装

  ### Windows

  - 安装 CA

    ```bash
    certutil -addstore -f Root ca.crt
    ```

  - 查看 CA

    ```bash
    certutil -store  Root
    ```

  - 删除 CA

    ```bash
    certutil -delstore -f  Root 证书指纹
    ```
