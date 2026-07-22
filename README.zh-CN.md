# Small Change Delivery

[English](README.md) | **简体中文**

一个面向 Codex 的轻量交付 Skill，用于在单个主会话、单个 writer 内完成需求边界明确的中小型代码改动。

它把容易遗漏的工程动作固化为一个紧凑闭环：先锁定范围和验收标准，再检查影响面、实施最小改动、按风险验证、简化代码，并完成 Review / Fix / Re-review。默认不 commit、不 push、不部署。

## 适用场景

适合以下任务：

- 小功能或局部行为调整
- 可稳定复现的 Bug 修复
- 单个接口、页面或模块的窄范围修改
- 不需要多阶段拆分、可在一个主会话内完成的低到中风险改动
- 希望在不过度设计的前提下，减少调用链、异常路径、测试和验证遗漏

典型调用：

```text
使用 $small-change-delivery 修复订单列表筛选条件失效的问题。

要求：
- 保持现有 API contract 不变
- 做最小范围修改
- 补回归测试
- 修复 Review 中成立的 P0-P3 后重新复查
- 不 commit、不 push
```

此 Skill 默认关闭隐式调用。请在任务中显式写出 `$small-change-delivery`。

## 不适用场景

出现以下情况时，不应强行使用本 Skill：

| 场景 | 推荐方式 |
| --- | --- |
| 需求或目标仍不清晰 | 先讨论方案、补 OpenSpec，或使用 `$grill-with-docs` |
| 超出一个会话的大型未知工作 | 使用 `$wayfinder` 建立决策地图 |
| 跨阶段、跨模块或需要隔离子会话交付 | 使用 `$subagent-delivery` |
| 资金、核心权限、安全边界、数据迁移、复杂并发或难回滚副作用 | 使用更严格的专项流程和独立 Review |
| 仅做只读 Review 或排错 | 直接使用对应的 Review / Diagnosis Skill |

## 工作流

```text
范围锁定
  ↓
影响面检查
  ↓
最小实现
  ↓
按风险验证
  ↓
代码简化
  ↓
Review → Fix → Re-review
  ↓
区分已验证、未验证与 Git 状态
```

### 交付优先级

优先完成已确认需求的最小生产可用闭环，确保主流程、必要异常路径、数据状态和接口行为可以共同验收。与该闭环不可分离的权限、数据一致性、事务、输入校验、敏感信息保护和失败恢复必须同步实现。

功能闭环成立后，再按实际风险补充常规安全、边界和回归处理。不为追求绝对安全、穷举理论边界或证明完全无 Bug 而扩大范围、增加无明确收益的抽象、覆盖低概率假设或无限延长 Review / Fix 循环。

### 1. 锁定范围

确认需求、验收标准、不做内容、风险、验证方式和发布权限。存在会改变业务规则、数据、权限、API 契约或范围的关键歧义时，暂停受影响部分，不把猜测写入生产代码。

### 2. 检查影响面

按任务需要追踪入口、Service、领域规则、Mapper/SQL、DTO/VO、前端字段、配置、迁移和现有测试。每个计划修改的行为都应明确数据来源、调用方、失败路径、兼容性和验证方式。

### 3. 实施最小改动

只修改验收标准要求的文件和行为，复用项目现有模式，不增加无必要的公共抽象、共享 helper 或配置，不顺手重构无关代码。

### 4. 按风险验证

先确认主流程、必要异常路径、数据状态和接口行为已经形成可验收闭环，再根据改动类型选择最小但充分的风险验证：

- Bug：优先增加可复现问题的回归测试
- 业务规则：覆盖正常、边界、非法状态和依赖失败
- API：覆盖参数校验、HTTP 状态、JSON contract、鉴权和异常映射
- SQL：覆盖查询条件、空结果、影响行数、事务和隔离
- 前端：覆盖主要交互、异步状态、错误态和关键请求参数
- 配置或文案：运行构建或静态检查，并记录可重复的手动验证

缺少环境、凭据或 fixture 时，明确标记 `BLOCKED` 或未验证，不用替代验证冒充真实验收。

### 5. 简化和复核

检查最终 diff，删除本次新增的重复、死代码、无效防御和过度抽象。随后围绕固定 diff 做 Review，修复所有成立、可执行、属于当前范围并影响验收或真实风险的 P0-P3，重跑相关验证，并针对新的 fixed point 重新 Review。超出范围、仅覆盖低概率理论场景或没有明确收益的建议作为残余风险记录，不据此扩大实现。

### 6. 汇报状态

最终结果区分：

- 完成的改动和关键文件
- 实际运行的验证命令、测试数量和结果
- Review fixed point、发现及修复结果
- HTTP、数据库、第三方系统等未验证项
- 剩余风险
- commit、push、deploy 状态

只有代码、适用验证和最终 Review 都成立时，才声明交付完成。

## 可选配合的 Skill

本 Skill 可以独立执行；存在对应能力时，也可按任务需要组合：

- `$code-simplifier`：收敛本次新增的复杂度
- `$code-review`：对中等风险或 API、SQL、事务、权限改动做独立审查
- `$diagnosing-bugs`：处理难复现或原因不明确的 Bug
- `$webapp-testing`：验证浏览器页面和交互行为
- `$asana-openspec-java-workflow:java-test-strategy`：为 Java/Spring/MyBatis 改动选择测试层级

这些 Skill 是按风险选择的增强项，不是每次执行都必须全部调用。

## 安装

将仓库克隆到 Codex 的用户 Skill 目录：

```bash
mkdir -p ~/.codex/skills
git clone https://github.com/lcw363/small-change-delivery.git ~/.codex/skills/small-change-delivery
```

安装后重启 Codex 或新建任务，使 Skill 清单重新加载。

### 更新

```bash
git -C ~/.codex/skills/small-change-delivery pull --ff-only
```

### 卸载

移除 `~/.codex/skills/small-change-delivery` 目录，然后重启 Codex。删除前如需保留本地修改，请先备份或提交。

## 目录结构

```text
small-change-delivery/
├── SKILL.md
├── README.md
├── README.zh-CN.md
└── agents/
    └── openai.yaml
```

- `SKILL.md`：Codex 实际执行的流程和门禁
- `README.md`：英文安装、选型和使用说明
- `README.zh-CN.md`：简体中文安装、选型和使用说明
- `agents/openai.yaml`：Skill 列表中的展示名称、简介和默认提示词

## 设计原则

- 小改动使用小流程，不创建不必要的计划地图或多阶段流水线
- 先证据后结论，不根据文件名或表面现象猜测调用链
- 最小生产可用修改，不扩大需求边界
- 测试强度与风险匹配，不机械追求全量测试
- Review 结论绑定最终 fixed point，修复后必须重新复查
- 代码完成、测试通过、真实验收、提交和发布分别汇报

## 反馈与贡献

如果发现某类中小改动仍容易遗漏，请提交 Issue，并尽量提供：

- 任务类型和技术栈
- 原始请求示例
- 被遗漏的步骤或风险
- 期望的完成标准

修改流程规则时，应优先更新 `SKILL.md`；README 只保留用户需要理解的安装、选型和使用信息。
