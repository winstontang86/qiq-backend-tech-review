[new file]
# qiq-backend-tech-review

> 互联网后台技术方案的多维度系统化评审 Skill。

当前版本：**0.2.0**（版本号定义在 [SKILL.md](./SKILL.md) frontmatter 的 `version` 字段，作为打包与报告写入的**单一来源**）。

## 它能做什么

面向后台 / 分布式 / 微服务 / 数据系统 / 高并发系统的技术方案评审场景，按统一流程做 8 大维度系统化检查，并产出可仲裁的严重程度与上线门禁结论，避免"只看到自己关心的那一面"。

- **覆盖 8 大维度**：正确性与一致性、容错与高可用、性能与扩展性、安全、数据与存储、可运维性、简洁性、API 契约与兼容性。
- **两种执行模式**：
  - 正式评审模式（默认）：缺关键上下文必须追问，输出"是否可上线"门禁结论。
  - 初步扫描模式：允许带假设快速过一遍，仅输出风险清单，不下门禁结论。
- **统一严重程度与冲突仲裁**：跨维度发现按统一标准定级（Blocker / High / Medium / Low / Info），冲突项给出仲裁建议。
- **结构化产出**：基于 [templates/final-report.md](./templates/final-report.md) 输出最终报告。

## 何时使用 / 不适用

启用条件（任一即可）：

- 给出后台 / 分布式 / 微服务 / 数据系统 / 高并发系统的方案 / 设计文档 / RFC，希望"评审 / review / 挑刺 / 找漏洞 / 上线前检查"。
- 出现关键词：技术方案评审 / 架构评审 / 设计评审 / design review / RFC review / 上线前评审 / 技术方案校验。
- 希望对方案做**多维度系统化检查**，而非只问单点（如只问性能）。

不适用：

- 仅纠结一个具体技术问题（如"Redis 为什么慢"）。
- 前端 / 移动端 / 算法模型方案。
- 业务需求 / PRD 评审（本 skill 关注技术实现）。

## 评审产物（重要）

每次评审都会在与原方案文件**同目录**输出一个评审报告：

```
方案路径：docs/order-system.md
↓
报告路径：docs/order-system-review.md
```

### 重跑策略：先备份、再覆盖

当对**同一个方案**重新跑评审、目标 review 文件已存在时：

1. 先把旧报告**重命名**为 `<原文件名>-review.bak.YYYYMMDD-HHmmss.md`，保留在同一目录作为备份。
2. 再用本次结果**覆盖写**到稳定路径 `<原文件名>-review.md`。
3. 新报告头部会写出 `上一版备份路径` 字段，便于 diff 与回滚。

> 设计动机：让"最新一次评审结果"始终位于一个**稳定路径**（便于在 PR / Wiki 上长期引用），同时不丢历史。

示例：

```
首次评审：
  docs/order-system.md
  → 生成 docs/order-system-review.md

第二次重跑（2026-05-21 21:57:08）：
  docs/order-system-review.md           ← 旧报告
  → 重命名为 docs/order-system-review.bak.20260521-215708.md（备份）
  → 覆盖写入 docs/order-system-review.md（最新）
```

仅当用户**明确表达"不需要备份"**时跳过第 1 步。

### 报告头部元数据

每份报告头部包含可追溯的 5 项元信息：

- 评审时间（到秒）
- skill 版本号（取自 [SKILL.md](./SKILL.md) frontmatter）
- 原方案路径
- 报告输出路径
- 上一版备份路径（首次跑填"无"）

## 目录结构

```
.
├── SKILL.md                       # Skill 入口，frontmatter 含 version
├── README.md                      # 本文件
├── references/                    # 各维度 checklist 与判定规则
│   ├── 00-context-template.md
│   ├── 01-consistency-completeness.md
│   ├── 02-fault-tolerance-ha.md
│   ├── 03-performance-scalability.md
│   ├── 04-security.md
│   ├── 05-data-storage.md
│   ├── 06-operability.md
│   ├── 07-simplicity.md
│   ├── 08-orchestration.md
│   ├── 09-severity-and-gate.md
│   └── 10-api-contract-compatibility.md
├── templates/
│   └── final-report.md            # 评审报告模板（决定落盘内容结构）
├── build.sh                       # 打包脚本：产出可分发的 zip（带版本号）
└── LICENSE
```

入口为 [SKILL.md](./SKILL.md)；具体校验规则按需加载 [references/](./references/) 下对应文件，避免一次性灌入全部上下文。

## 使用方式

### 1. 在支持 Skills 的 Agent 平台

将本仓库（或下面打包出的 zip）作为 Skill 导入即可。Agent 命中 [SKILL.md](./SKILL.md) 中的 "When to use" 后会自动按流程执行：先确认执行模式 → 收集上下文 → 分维度评审 → 仲裁与定级 → 产出最终报告（按上面的命名 / 备份 / 覆盖规则落盘）。

### 2. 本地直接当作 Prompt 资料

不依赖 Skill 平台时，也可以把 [SKILL.md](./SKILL.md) 作为系统提示，结合 `references/` 中的 checklist 做人工或 LLM 评审。

## 打包分发

仓库自带打包脚本，默认产物文件名带版本号：

```bash
# 打包到 dist/qiq-backend-tech-review-v<version>.zip
# 版本号自动取自 SKILL.md frontmatter（单一来源）
./build.sh

# 自定义输出路径
./build.sh -o /tmp/my-skill.zip
```

打包内容**只包含 skill 运行所需的 3 项**：`SKILL.md` / `references/` / `templates/`（白名单方式）。其他根目录文件（README、LICENSE、build.sh 自身、`dist/`、`.git/`、IDE 配置等）天然不会进入产物。

依赖：`bash` + `zip`（macOS / 主流 Linux 默认自带）。

## 版本号管理

- **版本号唯一来源**：[SKILL.md](./SKILL.md) frontmatter 的 `version` 字段。
- **采用语义化版本（SemVer）**：`MAJOR.MINOR.PATCH`：
  - MAJOR：不兼容的清单 / 流程结构变更（例如新增/删除维度、Output Format 重构）。
  - MINOR：向后兼容的清单条目新增、判例库扩充、模板字段新增。
  - PATCH：措辞修订、文案优化、错别字修复。
- **变更流程**：修改后请同步更新 `version`；`build.sh` 与报告头部会自动读取，无需手动改第二处。

## 维护说明

- 修改流程 / 触发条件：编辑 [SKILL.md](./SKILL.md)（同步 bump version）。
- 新增 / 调整某维度 checklist：编辑 [references/](./references/) 中对应文件；维度间协同与冲突仲裁规则在 `08-orchestration.md`，定级与门禁在 `09-severity-and-gate.md`。
- 调整最终报告结构：编辑 [templates/final-report.md](./templates/final-report.md)。
- 调整打包内容 / 排除规则：编辑 [build.sh](./build.sh) 中的 `REQUIRED` 与 `EXCLUDES`。

## License

见 [LICENSE](./LICENSE)。
