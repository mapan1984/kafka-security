# 3.4 JAAS 配置方式

Kafka 使用 JAAS 进行 SASL 配置。

## broker 端

broker 可以通过指定 JAAS 文件位置（`-Djava.security.auth.login.config=`）或者 `sasl.jaas.config` 配置 JAAS。

### JAAS 文件

JAAS 文件中 `KafkaServer` section 提供 broker 端的 SASL 配置。如果有多个 SASL listener，可以在 section name 之前增加 listener name 的小写拼写作为前缀，以此区分不同 listener 的配置，例如 `sasl_ssl.KafkaServer`。

JAAS 文件中 `Client` section 提供连接 zookeeper 服务的 SASL 配置，可以通过在 system property 指定 `zookeeper.sasl.clientconfig` 指定 section name，例如 `-Dzookeeper.sasl.clientconfig=ZkClient`。

```
KafkaServer {
 org.apache.kafka.common.security.plain.PlainLoginModule required
 username="admin"
 password="admin_pass"
 user_admin="admin_pass"
};


// 连接 zookeeper
Client {
   org.apache.zookeeper.server.auth.DigestLoginModule required
   username="zk_admin"
   password="zk_admin_pass";
};
```

### sasl.jaas.config 配置

另外，broker 同样可以使用 `sasl.jaas.config` 配置代替 JAAS 文件进行配置，这项配置的格式为 `listener.name.{listenerName}.{saslMechanism}.sasl.jaas.config`，如果有多种 saslMechanism，可以通过该配置进行区分，例如：


``` jproperties
listener.name.sasl_ssl.scram-sha-256.sasl.jaas.config=org.apache.kafka.common.security.scram.ScramLoginModule required \
    username="admin" \
    password="admin-secret";
listener.name.sasl_ssl.plain.sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required \
    username="admin" \
    password="admin-secret" \
    user_admin="admin-secret" \
    user_alice="alice-secret";
```

### 优先级

broker SASL 3 种配置方式的优先级为：

1. 首先查看 broker 配置文件中的 `listener.name.{listenerName}.{saslMechanism}.sasl.jaas.config` 配置
2. 如果没有，则查看 JAAS 配置文件中的 `{listenerName}.KafkaServer` secion
3. 如果没有，则查看 JAAS 配置文件中的 `KafkaServer` section

连接 zookeeper 的配置只能放在 JAAS 配置文件中。

## client

client 可以通过指定 JAAS 文件位置（`-Djava.security.auth.login.config=`）或者 `sasl.jaas.config` 配置 JAAS。

### JAAS 文件

JAAS 文件中 `KafkaClient` section 提供 client 端的 JAAS 配置，启动 client 时通过 JVM 参数指定 JAAS 文件位置，例如 ` -Djava.security.auth.login.config=/etc/kafka/kafka_client_jaas.conf`

### `sasl.jaas.config` 配置

``` jproperties
sasl.jaas.config=org.apache.kafka.common.security.scram.ScramLoginModule sufficient username='admin' password='admin-secret';
```
