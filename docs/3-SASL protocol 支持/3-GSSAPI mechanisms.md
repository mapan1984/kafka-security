# 3.3 GSSAPI mechanisms

## Kerberos

### principal

principal 表示认证实体，由 3 部分组成(`name/instance@realm`)：

* name/primary
* instance
* realm

* keytab: 是一个存储在 client 或 service 端的文本文件，包含了 kerberos principal 和该 principal 的 master key
* ticket cache: client 与 kdc 交互完成后，包含身份认证信息的文件，短期有效，需要不断 renew

### kdc/client/service

kdc 数据库存储了它所有用户的 entry，我们用 principal 来引用一个 entry

## broker 配置

### 创建 principal

在 kdc 中为每个 broker 创建 principal，并生成 keytab 文件

``` bash
# 添加 principal
kadmin.local -q 'addprinc -randkey kafka/{hostname}@{REALM}'

# 生成 keytab 文件
kadmin.local -q "ktadd -k /etc/security/keytabs/{keytabname}.keytab kafka/{hostname}@{REALM}"
```

### kafka_server_jaas.conf

需要注意每个 broker 使用自己的 principal，JAAS 文件中 `keyTab` 与 `principal` 应该每个 broker 都不同。

``` conf
KafkaServer {
    com.sun.security.auth.module.Krb5LoginModule required
    useKeyTab=true
    storeKey=true
    keyTab="/etc/security/keytabs/kafka_server.keytab"
    principal="kafka/kafka1.hostname.com@EXAMPLE.COM";
};

// Zookeeper client authentication
Client {
    com.sun.security.auth.module.Krb5LoginModule required
    useKeyTab=true
    storeKey=true
    keyTab="/etc/security/keytabs/kafka_server.keytab"
    principal="kafka/kafka1.hostname.com@EXAMPLE.COM";
};
```

### 将 JAAS 和 krb5 文件位置（可选的）作为 JVM 参数

``` bash
-Djava.security.krb5.conf=/etc/kafka/krb5.conf
-Djava.security.auth.login.config=/etc/kafka/kafka_server_jaas.conf
```

### server.properties

监听地址：

``` jproperties
listeners=SASL_PLAINTEXT://host.name:port
```

SASL 使用的机制:

``` jproperties
sasl.enabled.mechanisms=GSSAPI
```

配置服务器名称，应与 broker 的 principal name 匹配

``` jproperties
sasl.kerberos.service.name=kafka
```

broker 之间如果要使用 SASL 进行通信，需要设置：

``` jproperties
security.inter.broker.protocol=SASL_PLAINTEXT
sasl.mechanism.inter.broker.protocol=GSSAPI
```

## 客户端配置

### 创建 principal

客户端需要在 kdc 中创建自己的 principal

### client.properties

``` jproperties
security.protocol=SASL_PLAINTEXT
sasl.kerberos.service.name=kafka
sasl.mechanism=GSSAPI
```

#### sasl.jaas.config

可以设置 `sasl.jaas.config` 参数指定 keytab 文件和 principal

``` jproperties
sasl.jaas.config=com.sun.security.auth.module.Krb5LoginModule required \
    useKeyTab=true \
    storeKey=true  \
    keyTab="/etc/security/keytabs/kafka_client.keytab" \
    principal="kafka-client-1@EXAMPLE.COM";
```

对命令行工具，如 `kafka-console-consumer` 或者 `kafka-console-producer`，使用 kinit 之后，如下：

    kinit -k -t /etc/security/keytabs/kafka_client.keytab kafka-client-1@EXAMPLE.COM

可以使用 `useTicketCache=true`

``` jproperties
sasl.jaas.config=com.sun.security.auth.module.Krb5LoginModule required \
    useTicketCache=true;
```

### kafka_client_jaas.conf

可用通过指定 JAAS 文件位置作为 JVM 参数，文件中 `KafkaClient` section 提供认证配置：

``` conf
KafkaClient {
    com.sun.security.auth.module.Krb5LoginModule required
    useKeyTab=true
    storeKey=true
    KeyTab="/home/hduser/keytabs/kafka.service.keytab"
    principal="kafka/node8@amanp.uom"
    useTicketCache=false
    serviceName=kafka;
};

Client {
    com.sun.security.auth.module.Krb5LoginModule required
    useKeyTab=true
    storeKey=true
    KeyTab="/home/hduser/keytabs/kafka.service.keytab"
    principal="kafka/node8@amanp.com"
    useTicketCache=false
    serviceName=kafka;
};
```

使用 kinit  之后可以使用 `useTicketCache=true`

``` conf
KafkaClient {
    com.sun.security.auth.module.Krb5LoginModule required
    useTicketCache=true
    renewTicket=true
    serviceName="kafka";
};
```

### 将 JAAS 和 krb5 文件位置（可选的）作为 JVM 参数

``` bash
-Djava.security.krb5.conf=/etc/kafka/krb5.conf
-Djava.security.auth.login.config=/tmp/kafka_client_jaas.conf
```

