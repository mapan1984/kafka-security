# 2.3 client 配置

## client_security.properties

``` jproperties
bootstrap.servers=180.76.140.179:9093

security.protocol=SSL

ssl.truststore.location=/root/client.truststore.jks
ssl.truststore.password=pass1984

# 如果 client 端也提供证书
# ssl.keystore.location=/root/client.keystore.jks
# ssl.keystore.password=pass1984
# ssl.key.password=pass1984

# server 地址与证书中的 CN 或者 SAN 中包含的信息一致
# 默认 https 表示需要验证，为空关闭验证
ssl.endpoint.identification.algorithm=

ssl.enabled.protocols = TLSv1.2,TLSv1.1,TLSv1
ssl.protocol = TLSv1.2
```

## 使用

    bin/kafka-console-consumer.sh --bootstrap-server 180.76.140.179:9093 --topic __test  --from-beginning --consumer.config client_security.properties
