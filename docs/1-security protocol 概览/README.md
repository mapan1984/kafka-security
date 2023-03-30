# 1 Security Protocol 概览

客户端连接时，是否需要认证以及通过什么方式认证，是否进行加密，由连接时使用的 security protocol 决定。

Kafka 提供了 4 种 security protocol 供选择，可以同时配置多个 listeners，同时支持多种协议接入。

* `PLAINTEXT`
* `SSL`
* `SASL_PLAINTEXT`
* `SASL_SSL`

| security.protocol | 用户认证                               | 传输加密 |
|-------------------|----------------------------------------|----------|
| PLAINTEXT         | 不需要进行认证                         | x        |
| SSL               | 可以开启客户端认证，需要客户端提供证书 | √        |
| SASL_PLAINTEXT    | 需要客户端提供用户信息                 | x        |
| SASL_SSL          | 需要客户端提供用户信息                 | √        |

是否开启权限控制不由 security protocol 决定，但是权限控制需要识别客户端连接的用户身份，只有在连接经过用户认证后才有意义

| security.protocol | 权限控制                                                             |
|-------------------|----------------------------------------------------------------------|
| PLAINTEXT         | 连接无用户身份，视为匿名用户 `ANONYMOUS`                             |
| SSL               | 如果开启客户端认证，则从客户端证书提取用户信息，可以支持开启权限控制 |
| SASL_PLAINTEXT    | 可以支持开启权限控制                                                 |
| SASL_SSL          | 可以支持开启权限控制                                                 |

### PLAINTEXT

* 不需要用户认证，任何客户端都可以连接服务
* 传输数据不加密
* 连接不携带用户身份，视为匿名用户（`KafkaPrincipal.ANONYMOUS`），可以创建用户名为 `ANONYMOUS` 的用户并绑定权限

| Broker-side                            | Client-side             |
|----------------------------------------|-------------------------|
| `listeners=PLAINTEXT://127.0.0.1:6667` |                         |

### SSL

client 与 broker 建立 SSL 连接，传输加密的数据：

* 每个 broker 需提供证书(keystore 文件)
* client 需要一个 truststore 文件，该 truststore 文件包含可信 ca 的证书，这里的 ca 就是签发 broker 证书的 ca

另外，如果 broker 想要通过 SSL 认证 client，则除上述要求，还需：

* broker 需要一个 truststore 文件，该 truststore 文件包含可信 ca 的证书，这里的 ca 就是签发 client 证书的 ca
* client 需要提供证书(keystore 文件)

这里的认证有 2 层含义：

1. 单向认证：kafka broker 提供证书，kafka client 需要验证 broker 提供的证书。验证内容包括：证书是否过期、是否可信（由信任的 ca 签发）、证书中域名是否与请求域名一致。
2. 双向认证：kafka broker 和 client 都需要提供证书，既需要 client 验证 broker 证书，也需要 broker 验证 client 证书。

如果需要对 client 连接设置 acl 规则，则需要从证书中提取用户信息，一般用证书的 CN 或者 SAN 信息

| Broker-side                      | Client-side             |
|----------------------------------|-------------------------|
| `listeners=SSL://127.0.0.1:6667` |                         |
| `inter.broker.protocol=SSL`      | `security.protocol=SSL` |

### SASL_PLAINTEXT

client 与 broker 之间通过 SASL 机制进行认证，传输内容不加密，可以为认证用户设置 acl 权限

| Broker-side                                                      | Client-side                                                     |
|------------------------------------------------------------------|-----------------------------------------------------------------|
| `listeners=SASL_PLAINTEXT://127.0.0.1:6667`                      |                                                                 |
| `inter.broker.protocol=SASL_PLAINTEXT`                           | `security.protocol=SASL_PLAINTEXT`                              |
| `sasl.mechanism=PLAIN / GSSAPI /  SCRAM-SHA-256 / SCRAM-SHA-512` | `sasl.mechanism=PLAIN / GSSAPI / SCRAM-SHA-256 / SCRAM-SHA-512` |

### SASL_SSL

client 与 broker 之间通过 SASL 机制进行认证，传输内容通过 SSL 加密，可以为认证用户设置 acl 权限

因为需要使用 SSL 加密，需要在 broker 配置 SSL 证书，client 是否携带证书不是必须的。

> 2.8.0 之后，SASL_SSL 可以同时开启 client 证书认证(KIP-684)

| Broker-side                                                     | Client-side                                                     |
|-----------------------------------------------------------------|-----------------------------------------------------------------|
| `listeners=SASL_SSL://127.0.0.1:6667`                           |                                                                 |
| `inter.broker.protocol=SASL_SSL`                                | `security.protocol=SASL_SSL`                                    |
| `sasl.mechanism=PLAIN / GSSAPI / SCRAM-SHA-256 / SCRAM-SHA-512` | `sasl.mechanism=PLAIN / GSSAPI / SCRAM-SHA-256 / SCRAM-SHA-512` |
