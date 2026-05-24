# RR Skill - Detailed Workflow

## Overview

RR Skill 通过四个阶段治理 AI 的代码修改行为：

1. **RR Analyze** - 分析问题，禁止改代码
2. **RR Commit** - 锁定计划，生成边界约束
3. **RR Implement** - 按约束执行修改
4. **RR Verify** - 检查边界合规性

---

## Important: V2 Limitations

**V2-alpha 是 Prompt Protocol，不是自动验证工具。**

- `rr verify` 输出依赖 AI 自查，**仍然需要人工审查**
- **PASS 不代表业务完全正确**，只代表边界检查通过
- **WARNING 必须人工验证 Unverified Items 后，再决定是否进入提交前人工审查**
- 如果存在 Unverified Items，状态不能是 PASS，只能是 WARNING 或 FAIL
- 未列入 Allowed 的新增文件一律 FAIL
- Implement 阶段禁止重新解释需求（检查 LAST_LOCKED_PLAN.md）

---

## Phase 1: RR Analyze

### Purpose

**禁止 AI 直接改代码。** 必须先进行结构化分析。

### Trigger

输入：`RR 分析` / `rr analyze` + 问题描述

示例：
```
RR 分析：用户登录后，session 状态未正确保存
```

### Process

```
用户输入问题
↓
AI 分析问题
↓
输出 ANALYZE_REPORT.md (DRAFT)
↓
用户纠正/补充
↓
AI 迭代更新 (v2, v3, ...)
↓
用户确认
↓
状态变为 LOCKED
↓
进入 Phase 2
```

### Rules

| Rule | Description |
|------|-------------|
| 禁止修改代码 | Analyze 阶段不允许修改任何文件 |
| 禁止生成 patch | 不输出代码片段或修改方案 |
| 禁止架构建议 | 不提出架构优化或重构建议 |
| 必须多轮校对 | 用户纠正后必须迭代更新 |

### Output: ANALYZE_REPORT.md

包含以下内容：

| Section | Description |
|---------|-------------|
| Status | DRAFT 或 LOCKED |
| Problem | 用户原始问题 |
| Classification | 问题类型分类 |
| Impact Radius | 涉及和不涉及的文件/模块 |
| Regression | 是否已知回归问题 |
| Risk Analysis | 可能误伤的模块 |
| Minimal Fix Strategy | 最小修改方案描述 |
| Version History | 多轮校对记录 |

### Escalate

如果问题无法在边界内解决：

AI 输出：
```markdown
## Analysis Result: ESCALATE

当前问题无法在限定边界内局部解决。

原因: [具体说明]

建议: 升级为 Architecture Issue
```

**禁止**：自动扩大修改范围

---

## Phase 2: RR Commit (LOCKED_PLAN)

### Purpose

将分析结果结构化沉淀为边界约束和任务包。

### Trigger

条件：ANALYZE_REPORT.md 状态为 LOCKED

输入：`确认计划` / `rr commit`

### Process

```
ANALYZE_REPORT.md (LOCKED)
↓
生成 BOUNDARY.md
↓
生成 PATCH_PLAN.md
↓
生成 TASK_PACKET.md
↓
进入 Phase 3
```

### Output Files

| File | Purpose |
|------|---------|
| BOUNDARY.md | Allowed/Forbidden Files/Behaviors/Stop Conditions |
| PATCH_PLAN.md | 本次 patch 的修改计划 |
| TASK_PACKET.md | 给 AI 执行的约束任务包 |

### LOCKED_PLAN Rules

一旦锁定：

| Rule | Description |
|------|-------------|
| 禁止重新解释需求 | 不允许在 Implement 阶段改变理解 |
| 禁止扩大 Allowed | 不允许增加新的 Allowed Files |
| 禁止减少 Forbidden | 不允许删除 Forbidden Files |
| 禁止新架构 | 不允许添加架构改进 |
| 禁止自由重构 | 不允许任何重构行为 |

### Long-Term Rules

是否写入 `.rr/rules/PATCH_RULES.md`：

- **写入**：重大/典型 regression，未来可能复现
- **不写入**：一次性 bug，无复现价值

只有用户明确要求时才写入长期规则。

---

## Phase 3: RR Implement

### Purpose

严格按 TASK_PACKET.md 约束执行修改。

### Trigger

输入：`开始执行` / `rr implement`

### Input Source

**只能**来自 `.rr/current/TASK_PACKET.md`

**禁止**：
- 自由发挥
- 参考 TASK_PACKET 以外的内容
- 重新解释需求

### Rules

| Rule | Description |
|------|-------------|
| 只修改 Allowed Files | 只允许修改边界指定的文件 |
| 禁止触碰 Forbidden Files | 禁止修改任何 Forbidden 文件 |
| 禁止 Forbidden Behaviors | 禁止重构、优化、架构改动等 |
| 遵守 Stop Conditions | 触达停止条件时立即停止 |

### Stop Conditions

| Condition | Action |
|-----------|--------|
| 需要修改 Forbidden Files | 立即停止，报告 |
| 问题无法在 Allowed 内解决 | 立即停止，报告 |
| 发现新 regression 风险 | 立即停止，报告 |
| 发现需求理解偏差 | 立即停止，报告 |
| 需要扩大修改范围 | 立即停止，报告 |

### Stop Output

```markdown
## STOP Report

停止原因: [具体说明]

当前状态:
- 已修改: [file list]
- 未完成: [remaining items]

建议: 返回 Phase 1 重新分析
```

---

## Phase 4: RR Verify

### Purpose

检查修改是否符合边界约束。

**重要提示**：V1 是 Prompt Protocol，`rr verify` 输出依赖 AI 自查，**仍然需要人工审查**。

### Trigger

输入：`RR 验证` / `rr verify`

### Hard Rules (v1.1)

1. **如果存在 Unverified Items，状态不能是 PASS，只能是 WARNING 或 FAIL**
2. **未列入 Allowed 的新增文件一律 FAIL**
3. **PASS 不代表业务完全正确**，只代表边界检查通过
4. **WARNING 必须人工验证 Unverified Items 后，再决定是否进入提交前人工审查**
5. **FAIL 必须停止，并根据风险决定回滚或返回 rr analyze**

### Process

```
检查修改文件列表
↓
检查新增文件列表
↓
比对 Allowed Files
↓
比对 Forbidden Files
↓
检查 Forbidden Behaviors
↓
统计 Unverified Items
↓
输出 VERIFY_REPORT.md
```

### Output: VERIFY_REPORT.md (v1.1)

| Section | Description |
|---------|-------------|
| Status | PASS / WARNING / FAIL |
| Modified Files | 实际修改的文件列表 |
| New Files Created | 新增文件及其 Allowed 状态 |
| Boundary Check | Allowed/Forbidden 检查结果 |
| Unverified Items | 尚未验证的内容（存在时不能 PASS） |
| Evidence | 修改证据记录 |
| Manual Verification Required | 需人工验证的项目 |
| Risks | 风险项列表 |
| Final Recommendation | 提交建议 |

### Status Definition (v2.0)

| Status | Condition | Action |
|--------|-----------|--------|
| PASS | 无越界、无 Forbidden、**无未验证关键项** | ✅ 边界检查通过，可进入提交前人工审查 |
| WARNING | 无越界、无 Forbidden，**但存在未验证项** | ⚠️ 必须人工验证 Unverified Items 后，再决定是否进入提交前人工审查 |
| FAIL | 触碰 Forbidden、超出 Allowed、违反 Locked Plan、或新增未 Allowed 文件 | ❌ 必须停止，并根据风险决定回滚或返回 rr analyze |

### Important Notes

- **PASS 不代表业务完全正确**，只代表边界检查通过，仍需人工审查
- **WARNING 必须人工验证 Unverified Items 后，再决定是否进入提交前人工审查**
- **FAIL 必须停止，并根据风险决定回滚或返回 rr analyze**

---

## Emergency Stop

### Trigger

输入：`停止，重新分析`

### Action

1. 立即停止当前操作
2. 回滚未提交的修改（如需要）
3. 返回 Phase 1
4. 重新生成 ANALYZE_REPORT.md (DRAFT)

---

## Directory Lifecycle (v2.0)

### .rr/rules/ (长期)

- 永久存在
- 跨 patch 复用
- 只有重大 regression 才写入（promote rule）
- 不是每个 bug 都写入

### .rr/current/ (当前 patch)

- 当前 patch 有效
- patch 结束后归档到 `.rr/archive/`
- 每次新 patch 开始时重新生成

### .rr/state/ (状态文件)

- 当前 patch 有效
- 包含 CURRENT_STATE.md（当前 RR 阶段）
- 包含 LAST_LOCKED_PLAN.md（防止 implement 重新解释需求）
- patch 结束后归档到 `.rr/archive/`

### .rr/archive/ (归档)

- 永久存在（可定期清理超过 30 天的归档）
- 存放已完成 patch 的历史文件
- 用于历史追溯和规则沉淀评估

---

## State Files (v2.0)

### CURRENT_STATE.md

记录当前 RR 流程的阶段状态。

**Phase Flow**：
```
none → analyze (DRAFT) → analyze (LOCKED) → commit → implement → verify → done
```

**硬规则**：禁止跳跃阶段，必须按顺序 transition。

### LAST_LOCKED_PLAN.md

保存最近一次锁定的 Analyze 结果。

**硬规则**：Implement 阶段禁止重新解释需求。

如需改变理解，必须 STOP 并返回 rr analyze。

---

## Promote Rule (v2.0)

**不是每个 bug 都写入长期规则。**

只有符合以下条件才 promote：
- Severity: High / Critical
- Pattern: Yes（典型错误模式）
- Recurrence Risk: High
- User Confirmation: Required

---

## Best Practices

### Analyze 阶段

1. 仔细检查 Impact Radius 是否正确
2. 确认 Regression Detection 是否准确
3. 验证 Risk Analysis 是否完整
4. 多轮校对直到确认无误

### Commit 阶段

1. 仔细检查 BOUNDARY.md 的 Allowed/Forbidden
2. 确认 Stop Conditions 是否覆盖关键风险
3. 验证 TASK_PACKET.md 是否清晰明确
4. **如需新增文件，必须提前列入 Allowed New Files**

### Implement 阶段

1. 只关注 TASK_PACKET.md 的内容
2. 随时关注 Stop Conditions
3. 有疑问立即停止，不要继续
4. **禁止新增未列入 Allowed 的文件**

### Verify 阶段 (v2.0)

1. 认真检查 VERIFY_REPORT.md
2. **PASS 不代表业务完全正确**，仍需人工审查
3. **WARNING 必须人工验证 Unverified Items 后，再决定是否进入提交前人工审查**
4. **FAIL 必须停止，并根据风险决定回滚或返回 rr analyze**
5. 检查 Unverified Items，存在时状态不能是 PASS

---

## Common Mistakes

| Mistake | Consequence | Prevention |
|---------|-------------|------------|
| 跳过 Analyze | 修改范围失控 | 坚持先分析后修改 |
| 单轮确认 | 边界不准确 | 多轮校对 |
| 忽略 Stop Conditions | 引入 regression | 立即停止并报告 |
| FAIL 后继续提交 | 破坏系统 | 必须停止，根据风险决定回滚或返回 rr analyze |

---

## Summary

| Phase | Key Principle |
|-------|---------------|
| Analyze | 禁止改代码，先分析 |
| Commit | 锁定计划，禁止变轨 |
| Implement | 只改 Allowed，遵守边界 |
| Verify | PASS 可进入人工审查；WARNING 必须验证后决定；FAIL 必须停止并根据风险决定 |