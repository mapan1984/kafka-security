# 3 SASL 支持

Kafka 提供 `SASL_PLAINTEXT` 与 `SASL_SSL` protocol，区别在于 `SASL_SSL` 会使用 SSL 加密数据传输，如果使用 `SASL_SSL`，则必须增加 SSL 配置

Kafka 支持以下 SASL 机制：

* GSSAPI (Kerberos) - 从 0.9.0.0 版本开始
* PLAIN - 从 0.10.0.0 版本开始: clients authenticate using username/pssword
* SCRAM-SHA-256/SCRAM-SHA-512 - 从 0.10.2.0 版本开始
* OAUTHBEARER - 从 2.0 版本开始

这里有一个容易混淆的点：

1. Kafka 关于 `SASL` 提供 2 种协议(`security.protocol`)，分别是 `SASL_PLAINTEXT` 与 `SASL_SSL`，区别在于是否使用 SSL 加密数据传输
2. `PLAIN` 是 `SASL` 的一种机制(`sasl.enabled.mechanisms`)，与数据传输是否加密无关，`SASL_PLAINTEXT` 与 `SASL_SSL` 都可以使用这一机制

