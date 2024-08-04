# 5.2 JAAS Login Configuration File

```
<entry name> { 
    <LoginModule> <flag> <LoginModule options>;
    <LoginModule> <flag> <LoginModule options>;
    . . .
};

<entry name> { 
    <LoginModule> <flag> <LoginModule options>;
    <LoginModule> <flag> <LoginModule options>;
    . . .
};
```

## LoginModule

`LoginModule` 需要实现 `javax.security.auth.spi.LoginModule` 接口，决定了该 entry 通过何种方式认证。

一个 entry 中可以配置多个 `LoginModule`，认证流程会按照顺序依次执行 `LoginModule`

## Flag

`javax.security.auth.login.AppConfigurationEntry.LoginModuleControlFlag`

- Required：要求对应的 LoginModule 成功，无论成功与否，都会继续判断后面的 LoginModule。
- Requisite：要求对应的 LoginModule 成功，如果成功，会继续判断后面的 LoginModule。
- Sufficient：如果对应 LoginModule 成功，则认证流程成功并返回，否则继续判断后面的 LoginModule。
- Optional：不要求对应的 LoginModule 成功，无论成功与否，都会继续判断后面的 LoginModule。

## LoginModule Options

可以包含多个 `key=value` 形式的额外配置

## 参考

- https://www.jianshu.com/p/4585ce68b2ab
- https://docs.oracle.com/javase/7/docs/api/javax/security/auth/login/Configuration.html
