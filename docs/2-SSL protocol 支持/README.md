# 2 SSL protocol 支持

> Enabling SSL may have a performance impact due to encryption overhead.

`SSL` protocol 既可以用作加密，也支持用作认证

要注意 broker 端和 client 端的 `ssl.protocol` 要统一，要不然可能会导致无法正常获取证书信息

## 参考

- https://docs.confluent.io/2.0.0/kafka/ssl.html
- https://docs.confluent.io/platform/current/kafka/authentication_ssl.html
- https://www.orchome.com/171
