# kafka 认证与鉴权

Kafka 作为数据服务，需要考虑以下 3 个问题：

1. **用户认证**：哪些客户端可以连接 Kafka 服务
2. **权限控制**：客户端连接 Kafka 服务之后，可以访问哪些资源（topic, consumer 等），做什么操作
3. **传输加密**：客户端与 Kafka 服务交互的数据是否会被第三方拦截窃听

Kafka 对这 3 个问题的解决方式为：

1. 用户认证 Authentication(SSL or SASL)：客户端连接服务时，提供自身身份信息，服务端验证通过后才准许客户端连接。支持的认证方式有：
    * SSL: 客户端提供自身 SSL 证书，服务端验证证书并从证书中提取身份信息。
    * SASL (SASL_PLAINTEXT or SASL_SSL): SASL 是一种客户端与服务端之间的认证机制规范，全称 Simple Authentication and Security Layer。kafka 支持以下 SASL 机制：
        * GSSAPI (Kerberos) - 从 0.9.0.0 版本开始
        * PLAIN - 从 0.10.0.0 版本开始: clients authenticate using username/pssword
        * SCRAM-SHA-256/SCRAM-SHA-512 - 从 0.10.2.0 版本开始
        * OAUTHBEARER - 从 2.0 版本开始
2. 权限控制 Authorisation(ACL)：可以设置规则，允许或者禁止哪些用户可以对哪些资源进行什么操作
3. 传输加密 Encryption(SSL)
