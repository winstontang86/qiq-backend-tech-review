# 仓库规则与禁用/弃用事项预扫描

> 本文件由 [SKILL.md](../SKILL.md) **Step 1.6** 调用，用于在正式校验前生成"**禁用清单**"。所有校验者（特别是校验者 1 §12、校验者 4、校验者 5、校验者 6、校验者 8）在审查时必须消费本清单的产出，不得脑补。

## 一、为什么单独有这一步

- "**违反仓库里面明确规定的不允许事项 / 弃用的公共方法**"是常见但难发现的翻车点：方案表面合理，但与团队多年沉淀的红线、最佳实践、统一框架冲突。
- 这类问题不应在每个校验者里各自从零搜索；应一次性扫描，输出禁用清单，供所有维度复用。
- 检索范围聚焦在"**仓库内已有书面规定**"，避免靠记忆 / 经验脑补出"我觉得不允许"。

## 二、检索范围（凭证据，不凭记忆）

下列位置全部检索一遍；命中条目越多，禁用清单越完整。

### 2.1 规则 / 规范文件

- 仓库根 / docs / .cursor / .codebuddy / .vscode 等目录下：
  - `*.cursorrules` / `*.cursor/rules/*.md`
  - `coding-standard*.md` / `code-style*.md` / `dev-guideline*.md` / `architecture*.md`
  - `SECURITY.md` / `CONTRIBUTING.md` / `STYLE_GUIDE*` / `BANNED_API*` / `DEPRECATED*`
  - 团队内部 wiki 链接（如方案中显式引用）
- 顶层 `README*.md` / `ARCHITECTURE*.md` / `RFC` 目录中**明确以"禁止 / 不允许 / forbidden / banned / deprecated / DO NOT USE / unsafe"**为关键词的段落。
- 提示词/会话注入的 `<rules>` 块（如本会话开头的 `security_rules`）。

### 2.2 代码与注释中的显式标记

- 关键字检索（不区分大小写）：
  - `deprecated` / `@Deprecated` / `Deprecated:` / `// DEPRECATED` / `# DEPRECATED`
  - `do not use` / `DO NOT CALL` / `禁用` / `已废弃` / `请勿使用` / `已弃用`
  - `unsafe` / `不可靠` / `unreliable` / `legacy`（在文档 / 公共方法注释中出现时）
- 公共组件 / SDK 仓库的发版说明、Migration Guide、CHANGELOG 中"**Removed / Banned / Replaced by**"段落。

### 2.3 工具配置中固化的红线

- linter / 静态扫描配置（`.eslintrc*` / `golangci.yml` / `pylintrc` / `checkstyle*.xml` / `sonar-project.properties` 等）中**显式禁用的规则 / 包 / API 名单**。
- 依赖管理白/黑名单（如 `dependency-check`、企业内部依赖治理平台导出的白名单清单）。
- CI 流水线中拦截规则（如禁止提交 `SELECT *`、禁止用 `System.out.println`、禁止 `time.Sleep` 在 prod 路径等）。

### 2.4 历史故障与复盘沉淀

- 仓库内 `incidents/` / `postmortem/` / `复盘/` 等目录中**明确给出"事故根因 → 后续禁止 XXX"**的条目。
- 架构组 / 安全组的红线说明（如"以后不允许在业务请求线程中做长耗时阻塞"）。

## 三、产出格式（禁用清单）

把上述检索结果归并到一份"**禁用清单**"中，作为本步的最终产出。所有命中条目按下列字段化表格组织：

| 编号 | 类别 | 条目（一句话） | 来源（文件路径 / 行号 / 链接） | 适用范围 | 严重度（默认） | 是否可豁免 |
|---|---|---|---|---|---|---|
| BAN-1 | 安全 | 禁止字符串拼接 SQL，必须用参数化查询 | `<rules>/security_rules` Rule 1 | 全仓库 | Blocker | 否 |
| BAN-2 | 公共方法 | `legacy.HttpUtil.post()` 已 Deprecated，使用 `commons.HttpClient` 替代 | `legacy/HttpUtil.java:23 // DEPRECATED` | 全仓库 | High | 否 |
| BAN-3 | 数据 | 金额字段不得使用浮点类型 | `docs/data-standard.md §3.2` | 资金 / 账务相关表 | Blocker | 否 |
| BAN-4 | 架构 | 业务服务不得直连他业务 DB，必须走对应业务 SDK | `ARCHITECTURE.md §6` | 业务服务 | Blocker | 否 |
| BAN-5 | 运维 | 禁止在请求线程中调用 `time.Sleep > 100ms` | `coding-standard.md §4.7` | Web 服务 | High | 是（需架构组审批） |

类别建议（任选其一即可，多类别可逗号分隔）：
- 安全 / 数据 / 架构 / 公共方法 / 性能 / 运维 / 合规 / 命名 / 兼容性

严重度默认值（汇总到最终报告时按 [@references/09-severity-and-gate.md](09-severity-and-gate.md) 归一化）：
- 涉及安全 / 资金 / 合规 / 数据丢失 → 默认 **Blocker**。
- 涉及公共方法替代 / 架构红线 / 性能红线 → 默认 **High**。
- 涉及命名 / 风格 / 内部约定 → 默认 **Medium / Low**。

## 四、空清单的处理

如果上述检索全部为空（仓库**无任何**书面规定的禁用 / 弃用条目）：

1. 必须在产物中**显式记录**："本仓库未检索到 cursor rules / coding standards / 安全规范 / 内部 wiki 中的禁用条目"。
2. 在每个相关校验者的"仓库禁用项命中清单"小节明确写"未检索到仓库规则文件 / 禁用清单为空"，**不允许静默跳过**。
3. 不得用"通用经验"代替"仓库书面规则"作为禁用判定依据；通用经验只能作为风险提示，不能升 Blocker。

## 五、与各校验者的接入点

| 维度 | 接入位置 | 说明 |
|---|---|---|
| 校验者 1 §12 | "仓库规则与禁用事项一致性" | 主入口；负责对账方案是否命中禁用清单 |
| 校验者 4 | §〇 NFR 对账 + Rules | 把"安全相关禁用项"作为 Blocker 直入 |
| 校验者 5 | §〇 关键存储齐备度 + Rules | 把"数据 / 字段类型相关禁用项"（如浮点存金额）直入 |
| 校验者 6 | 变更管理 / 配置 / 日志 | 把"运维 / 日志 / 配置相关禁用项"直入 |
| 校验者 8 | 兼容性 / 公共方法演进 | 把"deprecated 公共方法"作为 High+ 直入 |

> 同一条命中项可能在多个维度出现，**最终汇总报告时**统一在"仓库规则违反清单"小节集中呈现，避免重复但不应漏报。
