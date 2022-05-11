# 3.2 SCRAM mechanisms

> Salted Challenge Response Authentication Mechanism

与 PLAIN 机制类似，但在 zookeeper 中存储用户信息(默认在 `/config/users` 节点下)，因此能够动态添加用户，对密码使用 sha 进行加密

## broker 配置

### 创建 SCRAM 证书（设置用户名、密码）

    export ZK_CONNECT="$(hostname):2181"

增加用户：

    kafka-configs.sh --zookeeper ${ZK_CONNECT} --alter --add-config 'SCRAM-SHA-256=[iterations=8192,password=alice_pass],SCRAM-SHA-512=[password=alice_pass]' --entity-type users --entity-name alice

    kafka-configs.sh --zookeeper ${ZK_CONNECT} --alter --add-config 'SCRAM-SHA-256=[password=admin_pass],SCRAM-SHA-512=[password=admin_pass]' --entity-type users --entity-name admin

描述用户：

    kafka-configs.sh --zookeeper ${ZK_CONNECT} --describe --entity-type users --entity-name alice

删除用户：

    kafka-configs.sh --zookeeper ${ZK_CONNECT} --alter --delete-config 'SCRAM-SHA-512' --entity-type users --entity-name alice

### kafka_server_jaas.conf

如果 broker 之间通过 SASL 通信，必须设置 broker 与其他 broker 进行连接的用户名和密码

增加 `kafka_server_jaas.conf` 文件：

```
KafkaServer {
 org.apache.kafka.common.security.scram.ScramLoginModule required
 username="admin"
 password="admin_pass";
};
```

运行 kafka 服务时，需要将 JAAS 配置文件位置作为 JVM 参数：

    -Djava.security.auth.login.config=/usr/local/kafka/config/kafka_server_jaas.conf

可以修改 `bin/kafka-server-start.sh`，增加以下内容:

``` sh
if [[ -f /usr/local/kafka/config/kafka_server_jaas.conf ]]; then
  export KAFKA_OPTS='-Djava.security.auth.login.config=/usr/local/kafka/config/kafka_server_jaas.conf'
fi
```

### server.properties

监听地址，这里同时使用 `SASL_PLAINTEXT` 与 `SASL_SSL`，要注意使用 `SASL_SSL` 需要配置 SSL，与之前一致，这里不做说明

``` jproperties
listeners=SASL_PLAINTEXT://:9094,SASL_SSL://:9095
advertised.listeners=SASL_PLAINTEXT://192.168.16.5:9094,SASL_SSL://180.76.140.179:9095
```

SASL 使用的机制：

``` jproperties
sasl.enabled.mechanisms=SCRAM-SHA-256
# 这里也可以配置多个
# sasl.enabled.mechanisms=PLAIN,SCRAM-SHA-256,SCRAM-SHA-512
```

broker 之间如果要使用 SASL 进行通信，需要设置:

``` jproperties
# SASL_PLAINTEXT 或者 SASL_SSL
security.inter.broker.protocol=SASL_PLAINTEXT
# SCRAM-SHA-256 或者 SCRAM-SHA-512
sasl.mechanism.inter.broker.protocol=SCRAM-SHA-256
```

## 客户端配置

客户端连接时需要配置 `kafka_client_jaas.conf` 文件

```
KafkaClient {
  org.apache.kafka.common.security.scram.ScramLoginModule required
  username="alice"
  password="alice_pass";
};
```

运行客户端时 `kafka_client_jaas.conf` 文件位置需要 作为 JVM 参数：

    -Djava.security.auth.login.config=/etc/kafka/kafka_client_jaas.conf

可以执行以下命令设置环境变量 `KAFKA_OPTS`:

    export KAFKA_OPTS='-Djava.security.auth.login.config=/etc/kafka/kafka_client_jaas.conf'

这个环境变量会被 `kafka-run-class.sh` 脚本读取并在运行 Java 时当作 JVM 参数传入

<!--
可以修改 `bin/kafka-console-consumer.sh`

``` bash
exec $(dirname $0)/kafka-run-class.sh -Djava.security.auth.login.config=/etc/kafka/conf/kafka_client_jaas.conf kafka.tools.ConsoleConsumer "$@"
```
-->

增加 `client.properties` 文件：

``` jproperties
security.protocol=SASL_PLAINTEXT

## 如果使用 SASL_SSL，需要同时配置 SSL
# security.protocol=SASL_SSL
#
# ssl.truststore.location=/etc/kafka/client.truststore.jks
# ssl.truststore.password=kafka1234
# ssl.endpoint.identification.algorithm=
# ssl.enabled.protocols = TLSv1.2,TLSv1.1,TLSv1
# ssl.protocol = TLSv1.2

sasl.mechanism=SCRAM-SHA-512
```

运行 `kafka-console-consumer.sh`

    kafka-console-consumer.sh --bootstrap-server 192.168.16.5:9094 --topic __test --from-beginning --consumer.config client.properties

生产者同理：

    kafka-console-producer.sh --bootstrap-server 192.168.16.5:9094 --topic __test  --producer.config client.properties
