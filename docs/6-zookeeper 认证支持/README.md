# 6 zookeeper 认证支持

## 创建 JAAS login file

创建 `conf/zookeeper_server_jaas.conf` 文件，配置用户名和密码：

```
Server {
   org.apache.zookeeper.server.auth.DigestLoginModule required
   user_admin="password"
   user_alice="password";
};
```

运行 zookeeper 时需要将 jaas 文件路径加入到 JVM 参数中，可以在 `conf/zookeeper-env.sh` 中增加：

``` sh
export SERVER_JVMFLAGS="-Djava.security.auth.login.config=/usr/local/zookeeper/conf/zookeeper_server_jaas.conf"
```

## zoo.cfg

``` cfg
authProvider.1=org.apache.zookeeper.server.auth.SASLAuthenticationProvider
requireClientAuthScheme=sasl
jaasLoginRenew=3600000
```

## zkCli 访问

客户端通过 SASL 认证访问，创建 `conf/zookeeper_client_jaas.conf` 文件：

```
Client {
   org.apache.zookeeper.server.auth.DigestLoginModule required
   username="admin"
   password="password";
};
```

运行 client 时需要将 jaas 文件路径加入到 JVM 参数中，可以在 `conf/zookeeper-env.sh` 中增加：

``` sh
export CLIENT_JVMFLAGS="-Djava.security.auth.login.config=/usr/local/zookeeper/conf/zookeeper_client_jaas.conf"
```

## kafka 访问

增加 jaas 文件，如 `kafka_server_jaas.conf`，添加以下内容：

``` jaas
Client {
    org.apache.kafka.common.security.plain.PlainLoginModule required
    username="admin"
    password="password";
};
```

运行 kafka 时需要将 jaas 文件露肩加入到 JVM 参数中

在创建 znode 时开启 ACL

``` jproperties
zookeeper.set.acl=true
```

## acl 操作

设置 acl

    [zk: localhost:2181(CONNECTED) 6] setAcl /brokers sasl:bcekafka:rwcda,world:anyone:r

查看 acl

    [zk: localhost:2181(CONNECTED) 7] getAcl /brokers
    'sasl,'bcekafka
    : cdrwa
    'world,'anyone
    : r
