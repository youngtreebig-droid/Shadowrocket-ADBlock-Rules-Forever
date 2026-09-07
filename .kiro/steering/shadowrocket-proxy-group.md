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

## 既要 PROXY 默认、又要能选具体节点：用两级分组

```conf
手动选择 = select,policy-regex-filter=.*
YouTube = select,PROXY,手动选择,日本节点,美国节点,policy-select-name=PROXY
```

- 服务分组不带正则筛选，显式列出策略；`policy-select-name` 指定的默认项同时放在列表第一位，避免依赖单一机制。
- 全部服务共用一个 `手动选择` 分组，不要给每个服务生成专属的节点子分组——本仓库明确要求保持精简。
  代价是同时选中「手动选择」的服务会共享同一个节点；需要区分时才额外加分组。

## ⚠️ 真正的生成源是 release.yml，不是 lazy_group.conf

`.github/workflows/release.yml` 的 `Update lazy rules` 步骤会：删除 `lazy_group.conf` →
从 `LOWERTOP/Shadowrocket` 重新下载 → 用内嵌 awk 的硬编码字符串**整体覆写全部服务分组**。

因此**只改 `lazy_group.conf` 不起作用**，分组的任何改动必须同步改 release.yml 里 awk 的 `groups[...]` 映射，
否则会被下一次构建静默还原。改完用下面的方式验证两边一致：

```bash
curl -so /tmp/low.conf https://raw.githubusercontent.com/LOWERTOP/Shadowrocket/main/lazy_group.conf
# 从 release.yml 抽出 awk 程序后执行，再对比 [Proxy Group] 段与 lazy_group.conf 是否一致
```

（注意 `/tmp` 在多次命令调用之间不保留，下载与验证要放在同一次执行里。）

## 其他

- 策略名大小写不敏感：`[Rule]` 里写 `YOUTUBE` 能匹配 `[Proxy Group]` 中定义的 `YouTube`（上游原版即如此，不要为此改名）。
- `lazy.conf` 是无策略组版本，`[Proxy Group]` 段故意留空，改动只应落在 `lazy_group.conf`。
- 改完分组后建议校验：`[Rule]` 引用的策略、分组内引用的成员是否都已定义，以及是否存在自引用。
