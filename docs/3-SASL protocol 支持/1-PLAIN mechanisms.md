# 3.1 PLAIN mechanisms

## broker 配置

### server.properties

监听地址，这里同时使用 `SASL_PLAINTEXT` 与 `SASL_SSL`，要注意使用 `SASL_SSL` 需要配置 SSL，与之前一致，这里不做说明

``` jproperties
listeners=SASL_PLAINTEXT://:9094,SASL_SSL://:9095
advertised.listeners=SASL_PLAINTEXT://192.168.16.5:9094,SASL_SSL://180.76.140.179:9095
```

SASL 使用的机制：

``` jproperties
sasl.enabled.mechanisms=PLAIN
# 这里也可以配置多个
# sasl.enabled.mechanisms=PLAIN,SCRAM-SHA-256,SCRAM-SHA-512
```

broker 之间如果要使用 SASL 进行通信，需要设置:

``` jproperties
security.inter.broker.protocol=SASL_PLAINTEXT
sasl.mechanism.inter.broker.protocol=PLAIN
```

### kafka_server_jaas.conf

```
KafkaServer {
    org.apache.kafka.common.security.plain.PlainLoginModule required
    username="admin"
    password="admin_pass"
    user_admin="admin_pass"
    user_alice="alice_pass";
};
```

* `username` 和 `password` 是 broker 与其他 broker 建立连接时所使用的用户名和密码
* 配置中的 `user_admin="admin_pass"` 的规则是 `user_<username>="<password>"`，就是说配置了 2 个用户 `admin` 和 `alice`，密码分别是 `admin_pass` 和 `alice_pass`，broker 用这些信息来验证客户端的连接，包括来自其他 broker 的连接

运行 kafka 服务时，需要将 JAAS 配置文件位置作为 JVM 参数：

    -Djava.security.auth.login.config=/usr/local/kafka/config/kafka_server_jaas.conf

可以修改 `bin/kafka-server-start.sh`，增加以下内容:

``` sh
if [[ -f /usr/local/kafka/config/kafka_server_jaas.conf ]]; then
  export KAFKA_OPTS='-Djava.security.auth.login.config=/usr/local/kafka/config/kafka_server_jaas.conf'
fi
```

## 客户端配置

客户端连接时需要配置 `kafka_client_jaas.conf` 文件

```
KafkaClient {
  org.apache.kafka.common.security.plain.PlainLoginModule required
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
exec $(dirname $0)/kafka-run-class.sh \
    -Djava.security.auth.login.config=/etc/kafka/conf/kafka_client_jaas.conf \
    kafka.tools.ConsoleConsumer "$@"
```
-->

增加 `client.properties` 文件：

``` jproperties
security.protocol=SASL_PLAINTEXT

sasl.mechanism=PLAIN
```

运行 `kafka-console-consumer.sh`

    kafka-console-consumer.sh --bootstrap-server 192.168.16.5:9094 --topic __test --from-beginning --consumer.config client.properties

生产者同理：

    kafka-console-producer.sh --bootstrap-server 192.168.16.5:9094 --topic __test  --producer.config client.properties
