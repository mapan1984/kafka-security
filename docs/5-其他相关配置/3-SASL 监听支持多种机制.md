# 5.3 SASL 监听支持多种机制

## 同一个 SASL_PLAINTEXT/SASL_SSL 的监听支持多种机制

kafka sasl 支持多种机制，可以同时开启：

``` jproperties
sasl.enabled.mechanisms=PLAIN,SCRAM-SHA-256,SCRAM-SHA-512
```

同时可以针对 listener_name 和 sasl mechanisms 进行不同配置。

这里在 `kafka_server_jaas.conf` 中针对 SASL_CLIENT 与 EXT_CLIENT 分别进行配置：

```
sasl_client.KafkaServer {
    org.apache.kafka.common.security.scram.ScramLoginModule sufficient
    username="admin"
    password="admin_pass";

    org.apache.kafka.common.security.plain.PlainLoginModule sufficient
    username="admin"
    password="admin_pass"
    user_admin="admin_pass";
};

ext_client.KafkaServer {
    org.apache.kafka.common.security.scram.ScramLoginModule required
    username="admin"
    password="password";
};
```

client 通过 SASL_CLIENT 访问 broker 时，可以使用 3 种机制

``` jproperties
bootstrap.servers=127.0.0.1:9093

security.protocol=SASL_PLAINTEXT

sasl.mechanism=PLAIN
# sasl.mechanism=SCRAM-SHA-256
# sasl.mechanism=SCRAM-SHA-512

java.security.auth.login.config=./config/kafka_client_jaas.conf
```

```
KafkaClient {
    org.apache.kafka.common.security.plain.PlainLoginModule sufficient
    username="admin"
    password="password";

    // org.apache.kafka.common.security.scram.ScramLoginModule sufficient
    // username="admin"
    // password="admin_pass";
};
```
