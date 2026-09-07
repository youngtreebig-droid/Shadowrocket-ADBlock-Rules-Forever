# Shadowrocket [Proxy Group] 约束

## policy-regex-filter 与显式策略列表互斥

分组一旦带上 `policy-regex-filter`，Shadowrocket 只用「正则筛选出的节点」生成该分组的可选项，
写在类型后面的 `PROXY`、`DIRECT`、其他分组名会被**全部忽略**。

表现：点开分组后只有一串节点，`PROXY` / `DIRECT` 不出现，`policy-select-name=PROXY` 也随之失效。

以下写法都**不成立**，不要再尝试（本仓库已试错多次）：

```conf
YouTube = select,PROXY,policy-select-name=PROXY,policy-regex-filter=.*
YouTube = select,PROXY,use=true,policy-regex-filter=.*,policy-select-name=PROXY
YouTube = select,PROXY,include-all-proxies=true,policy-select-name=PROXY   # include-all-proxies 不是 Shadowrocket 参数
```

## 既要 PROXY 默认、又要能选单个节点：用两级分组

```conf
YouTube = select,PROXY,YouTube节点,日本节点,美国节点,policy-select-name=PROXY
YouTube节点 = select,policy-regex-filter=.*
```

- 服务分组不带正则筛选，显式列出策略；`policy-select-name` 指定的默认项同时放在列表第一位，避免依赖单一机制。
- 每个服务配一个独立的 `XX节点` 子分组，这样各服务的节点选择互不影响。共用一个子分组会导致改一个服务连带改掉其他服务。

## 其他

- 策略名大小写不敏感：`[Rule]` 里写 `YOUTUBE` 能匹配 `[Proxy Group]` 中定义的 `YouTube`（上游原版即如此，不要为此改名）。
- `lazy.conf` 是无策略组版本，`[Proxy Group]` 段故意留空，改动只应落在 `lazy_group.conf`。
- 改完分组后建议校验：`[Rule]` 引用的策略、分组内引用的成员是否都已定义，以及是否存在自引用。
