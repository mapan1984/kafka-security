# 4.3 定位 acl 规则

## acl 规则组成

- `AclBinding`
    - `ResourcePattern`
        - `ResourceType`: `enum` 类型，可选值有:
            - `UNKNOWN`: 代表任何不理解的类型
            - `ANY`: 在 filter 中，可以匹配任何类型
            - `TOPIC`
            - `GROUP`
            - `CLUSTER`
            - `TRANSACTIONAL_ID`
            - `DELEGATION_TOKEN`
        - `Resource Name`: topic 名，group id，transaction id 等。特别说明: 资源类型为 `CLUSTER`，Resource Name 固定为 `kafka-cluster`。
        - `PatternType`: `enum` 类型，可选值有：
            - `UNKNOWN`: 代表任何不理解的类型
            - `ANY`: 在 filter 中，可以匹配任何类型
            - `MATCH`: 在 filter 中，任何能匹配到其的资源，比如 `ResourcePatternFilter(TOPIC, "payments.received", MATCH)`，以下 3 个都能匹配：` ResourcePattern(TOPIC, "payments.received", LITERAL)`, `ResourcePattern(TOPIC, "*", LITERAL)`, `ResourcePattern(TOPIC, "payments.", PREFIXED)`
            - `LITERAL`: 全匹配，此类型下如果 resource name 是 `*` 可以匹配任何名称的资源
            - `PREFIXED`: 前缀匹配
    - `AccessControlEntry`
        - principal: 模式为 `{principalType}:{principalName}` 的字符串，比如: `KafkaPrincipal.USER_TYPE + ":alice"`
        - host
        - `AclOperation`:
            - UNKNOWN
            - ANY
            - ALL
            - READ
            - WRITE
            - CREATE
            - DELETE
            - ALTER
            - DESCRIBE
            - CLUSTER_ACTION
            - DESCRIBE_CONFIGS
            - ALTER_CONFIGS
            - IDEMPOTENT_WRITE
        - `AclPermissionType`
            - UNKNOWN
            - ANY
            - DENY
            - ALLOW

