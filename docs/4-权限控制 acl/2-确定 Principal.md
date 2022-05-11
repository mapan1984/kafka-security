# 4.2 确定 Principal

## Principal (User) 怎么确定

### SSL

对于 SSL，user name 来自 client 证书的 `CN=writeuser,OU=Unknown,O=Unknown,L=Unknown,ST=Unknown,C=Unknown` 信息(the X.500 certificate distinguished name)。

可以通过 `ssl.principal.mapping.rules` 从证书的 distinguished name 中提取 short name

```
RULE:pattern/replacement/
RULE:pattern/replacement/[LU]
```

如果有以下规则：

```
RULE:^CN=(.*?),OU=ServiceUsers.*$/$1/,
RULE:^CN=(.*?),OU=(.*?),O=(.*?),L=(.*?),ST=(.*?),C=(.*?)$/$1@$2/L,
RULE:^.*[Cc][Nn]=([a-zA-Z0-9.]*).*$/$1/L,
DEFAULT
```

那么从

    "CN=serviceuser,OU=ServiceUsers,O=Unknown,L=Unknown,ST=Unknown,C=Unknown"

会提取出 `serviceuser` 作为用户名(Principal)

从

    "CN=adminUser,OU=Admin,O=Unknown,L=Unknown,ST=Unknown,C=Unknown"

会提取出 `adminuser@admin` 作为用户名(Principal)

### SASL

Principal 即为注册的用户名

