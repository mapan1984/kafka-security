## JDK 11 keytool genkey 无法指定 keypass

    keytool \
        -genkey \
        -alias localhost \
        -keyalg RSA \
        -keysize 2048 \
        -keypass keypassword \
        -storepass keystorepassword \
        -keystore server.keystore.jks \
        -validity 36500 \
        -dname "CN=www.amanp.com, OU=amanp, O=amanp, L=BJ, ST=BJ, C=CN" \
        -ext SAN=DNS:localhost

JDK 11 创建 SSL key 告警：

> Warning:  Different store and key passwords not supported for PKCS12 KeyStores. Ignoring user-specified -keypass value.

解决方式：将 `keypass` 与 `storepass` 设置为相同值

- https://developer.android.com/studio/releases#ki-key-keystore-warning
- https://bugs.openjdk.java.net/browse/JDK-8008292

## 数字证书格式

* `*.DER` 或 `*.CER` 文件： 这样的证书文件是二进制格式，只含有证书信息，不包含私钥。
* `*.CRT` 文件： 这样的证书文件可以是二进制格式，也可以是文本格式，一般均为文本格式，功能与 `*.DER` 及 `*.CER` 证书文件相同。
* `*.PEM` 文件： 这样的证书文件一般是文本格式，可以存放证书或私钥，或者两者都包含。 `*.PEM` 文件如果只包含私钥，一般用 `*.KEY` 文件代替。
* `*.PFX` 或 `*.P12` 文件： 这样的证书文件是二进制格式，同时包含证书和私钥，且一般有密码保护。

## 格式转换

### JKS → PFX(PKCS12)

    keytool \
        -importkeystore \
        -srckeystore client.truststore.jks \
        -destkeystore client.truststore.pfx \
        -srcstoretype jks \
        -deststoretype pkcs12

### PFX → PEM/KEY/CRT

使用 OpenSSL 工具执行以下命令将 pfx 格式证书转换成 pem 证书文件：

    openssl pkcs12 -in client.truststore.pfx -nodes -out client.truststore.pem

>     openssl pkcs12 -in client.truststore.pfx -nokeys -out client.truststore.pem
>
>     openssl pkcs12 -in client.truststore.pfx -nocerts -out client.truststore.key -nodes
>
> * -nodes：一直对私钥不加密。
> * -nocerts：不输出任何证书。
> * -nokeys：不输出任何私钥信息值。

将 pem 证书文件转换为 key 格式密钥文件（server.key）与 crt 格式公钥文件（server.crt）

    openssl rsa -in client.truststore.pem -out client.truststore.key
    openssl x509 -in client.truststore.pem -out client.truststore.crt

- https://help.aliyun.com/document_detail/42214.html
