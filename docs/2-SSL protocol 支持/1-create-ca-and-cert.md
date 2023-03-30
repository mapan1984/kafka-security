# 2.0 创建 CA 和 Certificate

## 1. 创建 CA (certificate authority)

    openssl req -new -x509 -keyout ca-key -out ca-cert -days 36500

* ca-key: private and public key
* ca-cert: ca 自身的证书，自签名

## 2. 生成 truststore 文件

### 2.1 client.truststore.jks

将 ca 自身的证书 ca-cert 添加到 client.truststore.jks 中，客户端持有 client.truststore.jks 后可信任此 ca

    keytool -import -keystore client.truststore.jks -alias CARoot -file ca-cert

验证 truststore 内容：

    keytool -list -v -keystore client.truststore.jks [-storepass <storepass>]

    keytool -list -rfc -keystore client.truststore.jks [-storepass <storepass>]

### 2.2（可选）server.truststore.jks

如果需要双向认证，即让 kafka broker 验证 client 的证书，在 broker 端设置 `ssl.client.auth=required`。

同时 broker 端也需要 truststore 文件，包含签发 client 端证书的 ca 证书。

这里我们使用同一个 ca-cert：

    keytool -import -keystore server.truststore.jks -alias CARoot -file ca-cert

也可以按第一步重新创建新的 ca 用于签发 client 端证书。

## 3. 生成服务端证书

### 3.1 创建服务端 SSL key

为每个 Kafka broker 生成 keystore 文件，存储 privae key, public key 以及 certificate

    keytool \
        -genkey \
        -alias localhost \
        -keyalg RSA \
        -keysize 2048 \
        -keypass password \
        -storepass password \
        -keystore server.keystore.jks \
        -validity 36500 \
        -dname "CN=www.amanp.com, OU=amanp, O=amanp, L=BJ, ST=BJ, C=CN" \
        -ext SAN=DNS:localhost \

* -genkey/-genkeypair: 生成密钥对
* -alias localhost: 一个 keystore 文件可以存储多个 private key 与 certificate，以 alias name 标记
* -keyalg RSA: 指定密钥算法，生成公私钥（非对称加密，这里用 RSA）
* -keysize: 密钥 bit 长度
- -keypass: 密钥口令（密钥 password，使用密钥时需要，比如用密钥签名）
* -storepass password: keystore password 密钥库口令（对 keystore.jks 文件进行添加/查看等操作）
* -keystore server.keystore.jks: 生成的 keystore 文件
* -validity 36500: 有效期，天
* -dname "CN=www.sample.com, OU=sample, O=sample, L=BJ, ST=BJ, C=CN" \
    * CN(Common Name): first and last name
    * OU: organizational unit
    * O: organization
    * L: City or Locality
    * ST: State or Province
    * C: two-letter country code for this unit
* -ext SAN=DNS:{FQDN},IP:{IPADDRESS1}: SubjectAlternativeName 附加信息，与主机名验证有关

> 查看帮助
>
> keytool -genkey --help

> 其他参数：
>
> * sigalg: 签名算法（消息摘要）（SHA），与 keyalg 匹配，比如 keyalg 是 RSA，sigalg 可以用 SHA256withRSA

验证证书的内容：

    keytool -list -v -keystore server.truststore.jks [-storepass <storepass>]

    keytool -list -rfc -keystore server.truststore.jks [-storepass <storepass>]

#### 3.1.1 主机名验证(host name verification)

通过 SSL 协议连接验证证书时，会验证证书中的域名与实际连接的域名是否一致，以确保连接到真实的 server/client（这里的域名也可以是 hostname 或者 IP）。

证书包含的域名信息信息指创建创建证书时指定的 fully qualified domain name(FQDN) 或者 ip address，即下面 2 个 fields:

1. Common Name(CN)
2. Subject Alternative Name (SAN)

使用 Common Name 用于主机名验证从 2000 之后不再推荐，用 SAN field 更加灵活，允许使用 multiple DNS 和 IP entries 在证书中声明。

这个检查的主要目的是防止中间人攻击。很长时间以来，这个检查默认是关闭的，从 Kafka 2.0.0 之后，主机名验证默认开启。

服务端和客户端都可以通过将 `ssl.endpoint.identification.algorithm` 设置为空字符串关闭这个检查

或者针对特定的 listener 进行设置

    kafka-configs.sh --bootstrap-server localhost:9093 --entity-type brokers --entity-name 0 --alter --add-config "listener.name.internal.ssl.endpoint.identification.algorithm="

### 3.2 签名证书

使用 ca 签名服务证书

首先，从 keystore 文件中导出证书 server-cert，创建证书请求：

    keytool \
        -certreq \
        -storepass password \   # 使用生成server.keystore.jks 的密码
        -keystore server.keystore.jks \
        -alias localhost \
        -file server-cert

然后，用 ca 签名 server-cert，生成 server-cert-signed:

    openssl x509 -req \
        -CA ca-cert \
        -CAkey ca-key \
        -in server-cert \
        -out server-cert-signed \
        -days 36500 \
        -CAcreateserial \
        -passin pass:password  # 使用 ca 证书的密码

最后，需要将 ca 和 server 的证书导入 server.keystore.jks 文件

    keytool \
        -import \
        -noprompt \
        -storepass password \  # 使用生成 server.keystore.jks 的密码
        -keystore server.keystore.jks \
        -alias CARoot \
        -file ca-cert

    keytool \
        -import \
        -noprompt \
        -storepass password \  # 使用生成 server.keystore.jks 的密码
        -keystore server.keystore.jks \
        -alias localhost \
        -file server-cert-signed

* -noprompt: 不在提示是否信任证书，不需要输入 yes/no(相当于直接输入 yes 表示信任证书)
* -storepass: keystore 文件的密码

## 4.（可选）生成客户端证书

如果 server 需要验证 client 证书，则要给 client 签发证书，步骤与 server 端相同

创建客户端 SSL key 文件 client.keystore.jks

    keytool \
        -genkey \
        -alias localhost \
        -keyalg RSA \
        -keysize 2048 \
        -keypass password \
        -storepass password \
        -keystore client.keystore.jks \
        -validity 36500 \
        -dname "CN=www.amanp.com, OU=amanp, O=amanp, L=BJ, ST=BJ, C=CN" \
        -ext SAN=DNS:localhost \

签名证书

    keytool \
        -certreq \
        -storepass password \
        -keystore client.keystore.jks \
        -alias localhost \
        -file client-cert

    openssl x509 -req \
        -CA ca-cert \
        -CAkey ca-key \
        -in client-cert \
        -out client-cert-signed \
        -days 36500 \
        -CAcreateserial \
        -passin pass:password

将 ca 和 server 的证书导入 client.keystore.jks 文件

    keytool \
        -import \
        -storepass password \
        -keystore client.keystore.jks \
        -alias CARoot \
        -file ca-cert

    keytool \
        -import -storepass password \
        -keystore client.keystore.jks \
        -alias localhost \
        -file client-cert-signed
