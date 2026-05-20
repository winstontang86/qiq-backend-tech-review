---
name: backend-tech-review
description: 互联网后台系统技术方案的多维度校验工作流。当用户提交后台/分布式系统/微服务/高并发/数据存储/接口设计/架构设计等技术方案（设计文档、技术评审、架构评审、PRD 转 RFC、上线前评审）并希望进行系统化 review 时使用。覆盖一致性与完整性、容错与高可用、性能与可扩展性、安全性、数据一致性与存储、可运维性、简洁性（奥卡姆剃刀）七大维度，并给出冲突仲裁与最终修改建议。触发关键词：技术方案评审、架构评审、设计评审、design review、RFC 评审、技术方案校验、技术方案 review、上线前评审、技术评审检查。
description_zh: 后台技术方案评审
description_en: Backend tech design review
disable: false
agent_created: true
---

# backend-tech-review — 互联网后台技术方案多维校验

## When to use

满足任一条件就启用：

- 用户给出一份后台/分布式/微服务/数据系统/高并发系统的技术方案/设计文档/RFC，希望做"评审"、"review"、"挑刺"、"找漏洞"、"上线前检查"。
- 用户提到关键词：**技术方案评审 / 架构评审 / 设计评审 / design review / RFC review / 上线前评审 / 技术方案校验**。
- 用户希望对一份方案做 **多维度系统化检查**，而不是单一关注点提问（如只问性能）。

不适用：

- 仅纠结于一个具体技术问题（如"Redis 为什么慢"）——直接回答即可。
- 前端/移动端/算法模型方案——本 skill 维度不匹配。
- 业务需求评审、PRD 评审——本 skill 关注技术实现方案。

## Workflow

### Step 1 — 收集审查上下文（必须）

**禁止跳过此步直接 review**。先把上下文拉齐，否则校验深度会失真（核心系统和内部工具不能用同一把尺子）。

按 [@references/00-context-template.md](references/00-context-template.md) 收集：

- **业务**：重要性等级（核心/重要/一般）、业务领域、用户规模（DAU/MAU）、是否涉及资金。
- **技术**：团队规模、团队熟悉的技术栈、现有基础设施、上线时间窗。
- **规模**：当前 / 1 年后预期 QPS、当前 / 1 年后预期数据量。
- **SLA**：可用性目标、核心接口延迟（P99）、一致性要求、RPO、RTO、部署模式（单机房 / 同城双活 / 异地多活）。

如果方案中已写明，直接抽取并向用户复核；缺失项标注"未说明"，向用户追问关键缺项（至少要拿到：业务重要性 + 是否涉及资金 + 当前 QPS 量级 + SLA 目标）。

### Step 2 — 选择校验者组合

不要每次都跑全部 7 个校验者。按 [@references/08-orchestration.md](references/08-orchestration.md) 中的"分组策略"挑选：

**必选组（每个方案都跑）**

- 校验者 1：一致性与完整性 → [@references/01-consistency-completeness.md](references/01-consistency-completeness.md)
- 校验者 7：简洁性（奥卡姆剃刀）→ [@references/07-simplicity.md](references/07-simplicity.md)

**按方案特征挑选**

| 方案特征 | 加入的校验者 |
|---|---|
| 分布式系统 / 微服务 | 2（容错）+ 5（数据一致性） |
| 高并发 / 大数据量 | 3（性能）+ 5（数据一致性） |
| 面向公网 / 涉及用户数据 | 4（安全） |
| 已有系统改造 / 长期维护 | 6（可运维性） |
| 核心交易 / 资金系统 | 全部启用 |

**按业务重要性调档**

| 重要性 | 深度 |
|---|---|
| 核心系统 | 全部 7 个，严格模式 |
| 重要系统 | 必选组 + 2~3 个按需 |
| 一般系统 | 必选组 |
| 内部工具 / MVP | 仅校验者 1（轻量）+ 校验者 7 |

把"本次启用了哪几个校验者、为什么"先告诉用户，让他可以增减。

### Step 3 — 逐个校验者执行

对选中的每一个校验者：

1. 读取对应 reference 文件（见下表）。
2. **完全采用该 reference 中定义的 Role / Identity / 审查清单 / Output Format / Rules**——这是该校验者的 system prompt，不要自己改写或缩水。
3. 把 Step 1 收集的上下文注入到当前校验者的视角。
4. 逐项过审查清单，凡发现问题，按该校验者规定的 Output Format 输出。
5. 给出该维度的 **总结 + 评分（1–10）**。

| # | 校验者 | Reference |
|---|---|---|
| 1 | 一致性与完整性 | [@references/01-consistency-completeness.md](references/01-consistency-completeness.md) |
| 2 | 容错与高可用 | [@references/02-fault-tolerance-ha.md](references/02-fault-tolerance-ha.md) |
| 3 | 性能与可扩展性 | [@references/03-performance-scalability.md](references/03-performance-scalability.md) |
| 4 | 安全性 | [@references/04-security.md](references/04-security.md) |
| 5 | 数据一致性与存储 | [@references/05-data-storage.md](references/05-data-storage.md) |
| 6 | 可运维性 | [@references/06-operability.md](references/06-operability.md) |
| 7 | 简洁性（奥卡姆剃刀） | [@references/07-simplicity.md](references/07-simplicity.md) |

### Step 4 — 汇总仲裁

按 [@references/08-orchestration.md](references/08-orchestration.md) 的"冲突仲裁规则"做最终裁决：

**优先级排序**

1. 正确性（1）> 一切——逻辑矛盾/遗漏必须修，不论复杂度。
2. 安全性（4）> 简洁性——安全漏洞必须修。
3. 数据正确性（5）> 性能 > 简洁性——涉及钱的数据一致性不让步。
4. 需求驱动的健壮性（2）> 简洁性（7）——SLA 明确要求的容错不能砍。
5. 简洁性（7）> 非需求驱动的健壮/性能——非 SLA 要求的过度设计要砍。

**量化判断**

- 故障概率 × 影响 > 长期维护成本 → 采纳健壮性建议。
- 故障概率 × 影响 < 长期维护成本 → 采纳简洁性建议。

### Step 5 — 输出最终报告

按 [@templates/final-report.md](templates/final-report.md) 组织最终交付：

1. **审查上下文摘要**（Step 1 收集的）。
2. **本次启用的校验者列表 + 启用原因**。
3. **各校验者的发现**（按维度组织，每个问题保留原始编号如 `C-1`、`HA-3`、`SEC-2` 便于追溯）。
4. **冲突仲裁结论**（如果不同维度建议冲突）。
5. **必须修复 / 建议修复 / 可选优化** 三段式整改清单。
6. **整体评分汇总表 + 是否可进入下一阶段** 的明确结论。

## Pitfalls

- **跳过上下文收集** → 评审深度失真，对内部工具用核心系统的尺子是浪费；对核心系统用内部工具的尺子是失职。**必须执行 Step 1**。
- **跑全部 7 个校验者** → 上下文爆炸 + 用户淹没在噪声里。按方案特征裁剪，并向用户解释为何裁剪。
- **校验者输出格式不统一** → 后期难以汇总。严格遵守每个 reference 中规定的 Output Format（带编号、带证据、带建议）。
- **只列问题不给修改建议** → 没有交付价值。每个发现都必须包含具体修改建议；简洁性维度还必须给出"替代方案 + 失效条件"。
- **冲突时不仲裁** → 用户拿到的是矛盾的建议。Step 4 必须给出明确取舍。
- **凭印象写问题** → 一致性维度尤其要求"引用方案原文作为证据"，不能脑补。

## Verification

跑完后自检以下条目，全部满足才算合格交付：

- [ ] 已记录业务重要性 / 是否涉及资金 / QPS / SLA 这 4 项核心上下文。
- [ ] 已说明选了哪几个校验者及理由。
- [ ] 每个发现都包含：位置、严重程度、问题描述、原文证据、修改建议。
- [ ] 简洁性建议都包含：替代方案、前提条件、失效条件。
- [ ] 给出了"必须 / 建议 / 可选" 三段式整改清单。
- [ ] 给出了整体能否进入下一阶段的明确结论（不要"看情况"）。
