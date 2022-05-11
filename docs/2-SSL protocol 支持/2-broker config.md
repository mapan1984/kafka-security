# 2.2 broker 配置

## server.properties

监听地址，当对 client 暴露的地址与本地监听地址不一致时，使用 `advertised.listeners`：

``` jproperties
listeners=PLAINTEXT://:9092,SSL://:9093
advertised.listeners=PLAINTEXT://192.168.16.5:9092,SSL://180.76.140.179:9093
```

broker 内部连接使用的协议：

``` jproperties
security.inter.broker.protocol=PLAINTEXT
```

keystore 与 truststore 的位置与密码：

``` jproperties
# 只有需要验证客户端证书时需要设置
# ssl.truststore.location=/home/kafka/kafka.server.truststore.jks
# ssl.truststore.password=pass1984

ssl.keystore.location=/home/kafka/kafka.server.keystore.jks
ssl.keystore.password=pass1984

# 只有需要双向认证的时候需要设置
# ssl.key.password=pass1984
```

### 其他相关配置

SSL 版本：

``` jproperties
ssl.enabled.protocols = TLSv1.2,TLSv1.1,TLSv1
```

keystore 与 truststore 的文件格式：

``` jproperties
ssl.keystore.type = JKS
ssl.truststore.type = JKS
```

是否开启主机名验证：

``` jproperties
# 是否验证 broker 地址与证书中的 CN 或者 SAN 中包含的信息一致
# 默认 https 表示需要验证，为空关闭验证
# 当 broker 需要通过 SSL 连接其他 broker 时需要关注此项配置
ssl.endpoint.identification.algorithm=
```

### 验证客户端 certificate

是否需要验证 client certificate：

``` jproperties
# none, requested, required
ssl.client.auth=required
```

SSL listeners 可以配置 TLS client authentication (also known as mutual TLS authentication)，通过参数 `ssl.client.auth` 进行设置，可选的配置如下：

* `none` (default): 关闭 client authenticaion，assigns `User:ANONYMOUS` as `KafkaPrincipal`
* `requested`: enables optional client authentication, uses distinguished name (DN) from certificate as KafkaPrincipal by default if certificate provided,  `User:ANONYMOUS` otherwise.
* `required`: enables mandatory client authentication, uses DN from certificate as KafkaPrincipal by default.

Kafka 2.8.0 之前关闭 TLS client authentication for `SASL_SSL` listeners even if `ssl.client.auth` is configured.

2.8.0 之后，可以通过以下配置为 `SASL_SSL` listeners 配置 TLS client authentication

``` jproperties
# default
ssl.client.auth

# 对 ssl listener name 的配置，不配可以回退到 default
listener.name.<sslListenerName>.ssl.client.auth

# sasl_ssl listener name
listener.name.<saslListenerName>.ssl.client.auth
```

### 从客户端证书提取用户信息

从证书中提取用户信息：

``` jproperties
ssl.principal.mapping.rules
```

## 验证 SSL 端口

通过以下命令验证 SSL 端口

    openssl s_client -debug -connect 180.76.140.179:9093 -tls1

    openssl s_client -debug -connect 180.76.140.179:9093 -tls1_2

`-tls1`, `-tls1_2` 即对应 `TLSv1`, `TLSv1.2` protocol，有时候只有用特定的 protocol 才能正常访问，client 应该配置 `ssl.protocol` 与 server 端相匹配

在命令的输出中应该能看到证书信息，类似这个：

    -----END CERTIFICATE-----
    subject=C = cn, ST = beijing, L = beijing, O = amanp, OU = amanp, CN = mapan

    issuer=C = cn, ST = beijing, L = beijing, O = amanp, OU = amanp, CN = mapan, emailAddress = mapan@amanp.com

    ---

如果获取不到证书信息，则 SSL 访问不正常
