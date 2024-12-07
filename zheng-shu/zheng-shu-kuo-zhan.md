# 证书扩展



## **Basic Constraints**

基本约束,表示一个证书是否是CA证书

<img src="../.gitbook/assets/image (3) (1).png" alt="CA证书" data-size="original">![ssl证书](<../.gitbook/assets/image (8).png>)

`Path Length Constraint` 表示CA可签署的子CA层级数，默认None表示没有限制

**示例1**

<figure><img src="../.gitbook/assets/image (13).png" alt=""><figcaption></figcaption></figure>

**示例2**

<figure><img src="../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

**openssl config**

```
# 不是CA
basicConstraints = CA:FALSE
# 是 CA
basicConstraints = CA:TRUE
# 是CA并且可签署子CA层级1
basicConstraints = critical, CA:TRUE, pathlen:1
```



## **Subject Key Identifier 与** Subject Key Identifier



**Subject Key Identifier**

使用者密钥标识符,由证书的公钥hash计算而来

<figure><img src="../.gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure>



**Authority Key Identifier**

授权密钥标识符，上级CA的**Subject Key Identifier**

<figure><img src="../.gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>

**关系示例图**

<figure><img src="../.gitbook/assets/image (7) (1).png" alt=""><figcaption></figcaption></figure>

**openssl config**

```
# Subject Key Identifier
subjectKeyIdentifier = hash

# Authority Key Identifier
## keyid 尝试从颁发者证书复制使用者密钥标识符SKI
## issuer 当keyid 不存在 或者 使用always 属性，则会从颁发者证书复制颁发者 DN 和序列号
authorityKeyIdentifier = keyid, issuer
authorityKeyIdentifier = keyid, issuer:always
```

&#x20;

## Key Usage

密钥用法

<figure><img src="../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

| Name                | openssl            | Description                                                                                           |
| ------------------- | ------------------ | ----------------------------------------------------------------------------------------------------- |
| Digital signature   | `digitalSignature` | 当公钥与数字签名机制一起使用以支持Non-repudiation、Certificate signing 和 CRL signing以外的安全服务时使用。 数字签名常用于实体认证和数据来源的完整性认证。 |
| Non-repudiation     | `nonRepudiation`   | 当公用钥匙被用来验证用于提供不可抵赖服务的数字签名时使用。不可否认性可防止签署实体错误地否认某些行动（不包括证书或CRL签署）。                                      |
| Key encipherment    | `keyEncipherment`  | 当证书将与加密密钥的协议一起使用时使用。一个例子是S/MIME封装，其中使用证书中的公钥对快速（对称）密钥进行加密。SSL协议还执行密钥加密。                               |
| Data encipherment   | `dataEncipherment` | 当公钥用于加密用户数据（加密密钥除外）时使用。                                                                               |
| Key agreement       | `keyAgreement`     | 当公钥的发送方和接收方需要在不使用加密的情况下派生密钥时使用。然后，该密钥可以用于加密发送方和接收方之间的消息。密钥协商通常与Diffie-Hellman密码一起使用。                  |
| Certificate signing | `keyCertSign`      | 当主题公钥用于验证证书上的签名时使用。此扩展只能在CA证书中使用。                                                                     |
| CRL signing         | `cRLSign`          | 当主题公钥用于验证吊销信息（如CRL）上的签名时使用。                                                                           |
| Encipher only       | `encipherOnly`     | 仅在同时启用密钥协议时使用。这使得公钥仅用于在执行密钥协商时对数据进行加密。                                                                |
| Decipher only       | `decipherOnly`     | 仅在同时启用密钥协议时使用。这使得公钥仅用于在执行密钥协商时解密数据。                                                                   |



**openssl config**

可选择 `digitalSignature`, `nonRepudiation`, `keyEncipherment`, `dataEncipherment`, `keyAgreement`, `keyCertSign`, `cRLSign`, `encipherOnly`, and `decipherOnly`

<pre><code><strong>keyUsage = digitalSignature, nonRepudiation
</strong>keyUsage = critical, keyCertSign
</code></pre>



## Extended Key Usage

**增强型密钥用法**

<figure><img src="../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

| Name                                             | openssl         | 依赖的Key Usage                                                                                                         |
| ------------------------------------------------ | --------------- | -------------------------------------------------------------------------------------------------------------------- |
| SSL/TLS WWW Server Authentication                | serverAuth      | <ul><li><code>Digital signature</code></li><li><code>Key encipherment</code> or <code>Key agreement</code></li></ul> |
| SSL/TLS WWW Client Authentication                | clientAuth      | <ul><li><code>Digital signature</code> or <code>Key agreement</code></li></ul>                                       |
| Code Signing                                     | codeSigning     | <ul><li><code>Digital signature</code></li></ul>                                                                     |
| E-mail Protection (S/MIME)                       | emailProtection |                                                                                                                      |
| Trusted Timestamping                             | timeStamping    |                                                                                                                      |
| OCSP Signing                                     | OCSPSigning     |                                                                                                                      |
| ipsec Internet Key Exchange                      | ipsecIKE        |                                                                                                                      |
| Microsoft Individual Code Signing (authenticode) | msCodeInd       |                                                                                                                      |
| Microsoft Commercial Code Signing (authenticode) | msCodeCom       |                                                                                                                      |
| Microsoft Trust List Signing                     | msCTLSign       |                                                                                                                      |
| Microsoft Encrypted File System                  | msEFS           |                                                                                                                      |

**openssl**

```
extendedKeyUsage = critical, codeSigning, 1.2.3.4

extendedKeyUsage = serverAuth, clientAuth
```



## **Subject Alternative Name**

**使用者可选名称**

<figure><img src="../.gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>

| type      | 描述                                 |   |
| --------- | ---------------------------------- | - |
| email     | 邮件地址                               |   |
| URI       |                                    |   |
| DNS       | 域名                                 |   |
| RID       | a registered ID: OBJECT IDENTIFIER |   |
| IP        | ip地址                               |   |
| dirName   |                                    |   |
| otherName |                                    |   |



openssl

```
subjectAltName = email:copy, email:my@example.com, URI:http://my.example.com/

subjectAltName = IP:192.168.7.1

subjectAltName = IP:13::17

subjectAltName = email:my@example.com, RID:1.2.3.4

subjectAltName = otherName:1.2.3.4;UTF8:some other identifier

subjectAltName = DNS:www.zhxlp.com,DNS:*.zhxlp.com,IP:192.168.1.1

[extensions]
subjectAltName = dirName:dir_sect

[dir_sect]
C = UK
O = My Organization
OU = My Unit
CN = My Name
```



## Authority Info Access

**授权信息访问**

<figure><img src="../.gitbook/assets/image (2) (1).png" alt=""><figcaption></figcaption></figure>

| type             | 描述        | OID                |
| ---------------- | --------- | ------------------ |
| OCSP             | 联机证书状态协议  | 1.3.6.1.5.5.7.48.1 |
| caIssuers        | 证书颁发机构颁发者 | 1.3.6.1.5.5.7.48.2 |
| ad\_timestamping |           |                    |
| AD\_DVCS         |           |                    |
| caRepository     |           |                    |

**openssl config**

```
authorityInfoAccess = OCSP;URI:http://ocsp.example.com/,caIssuers;URI:http://myca.example.com/ca.cer

authorityInfoAccess = OCSP;URI:http://ocsp.example.com/

authorityInfoAccess = caIssuers;URI:http://myca.example.com/ca.cer
```



## **CRL distribution points**

**CRL分发点**

<figure><img src="../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>

**openssl config**

```
crlDistributionPoints = URI:http://example.com/myca.crl

crlDistributionPoints = URI:http://example.com/myca.crl, URI:http://example.org/my.crl
```



## Certificate Policies

**证书策略**

<figure><img src="../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

| OID            | 描述        |   |
| -------------- | --------- | - |
| 2.23.140.1.2.1 | DV        |   |
| 2.23.140.1.2.2 | OV        |   |
| 2.23.140.1.2.3 | IV        |   |
| 2.23.140.1.1   | EV        |   |
| 2.23.140.1.3   | EV代码签名    |   |
| 2.23.140.1.4.1 | 非 EV 代码签名 |   |

**openssl config**

```
certificatePolicies = 1.2.4.5, 1.1.3.4
```





## 参考

{% embed url="https://www.openssl.org/docs/man3.0/man5/x509v3_config.html" %}

{% embed url="http://www.pkiglobe.org/index.html" %}





