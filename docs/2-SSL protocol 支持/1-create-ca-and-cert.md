# 2.0 创建 CA 和 Certificate

## 1. 创建 SSL key 和 Certificate

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

    keytool -list -v -keystore server.keystore.jks

### 主机名验证

host name verification 是指从证书获取 server 提供的信息与正在连接的 server 端的 hostname 或者 ip 比较，以确保连接到真实的 server。

这个检查的主要目的是防止中间人攻击。很长时间以来，这个检查默认是关闭的，从 Kafka 2.0.0 之后，host name verification 默认开启。

可以通过将 `ssl.endpoint.identification.algorithm` 设置为空字符串关闭这个检查。

    bin/kafka-configs.sh --bootstrap-server localhost:9093 --entity-type brokers --entity-name 0 --alter --add-config "listener.name.internal.ssl.endpoint.identification.algorithm="

如果 host name verification 开启，clients 会确认 Server 的 fully qualified domain name(FQDN) 或者 ip address，即下面 2 个 fields:

1. Common Name(CN)
2. Subject Alternative Name (SAN)

尽管 Kafak 检查这 2 个 fields，使用 Common Name 用于 host name verification 从 2000 之后是不推荐的。用 SAN field 更加灵活，允许使用 multiple DNS 和 IP entries 声明在 certificate 中

## 2. 创建你自己的 CA (certificate authority)

    openssl req -new -x509 -keyout ca-key -out ca-cert -days 36500

* ca-key: private and public key
* ca-cert: ca 自身的证书，自签名

将 ca 自身的证书 ca-cert 添加到 client.truststore.jks 中，客户端持有 client.truststore.jks 后可信任此 ca

    keytool -import -keystore client.truststore.jks -alias CARoot -file ca-cert

验证 truststore 内容：

    keytool -list -v -keystore client.truststore.jks

### kafka broker require client authentication

如果需要让 server 验证 client 的证书，即在 broker 端设置 `ssl.client.auth=requested` 或者 `ssl.client.auth=required` 时，必须提供包含签发 client 端 certificate 的 CA 的 truststore 文件

    keytool -import -keystore server.truststore.jks -alias CARoot -file ca-cert

## 3. 签名证书

用 *步骤2* 生成的 ca 签名 *步骤1* 生成的证书

首先，从 keystore 文件中导出证书 server-cert，创建证书请求：

    keytool \
        -certreq \
        -storepass password \
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
        -passin pass:password

最后，需要将 ca 和 server 的证书导入 server.keystore.jks 文件

    keytool \
        -import \
        -noprompt \
        -storepass password \
        -keystore server.keystore.jks \
        -alias CARoot \
        -file ca-cert

    keytool \
        -import \
        -noprompt \
        -storepass password \
        -keystore server.keystore.jks \
        -alias localhost \
        -file server-cert-signed

* -noprompt: 不在提示是否信任证书，不需要输入 yes/no(相当于直接输入 yes 表示信任证书)
* -storepass: keystore 文件的密码

如果 server 需要验证 client 证书，则同样要给 client 签发证书，步骤与 server 端相同
