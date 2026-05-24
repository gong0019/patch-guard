# PatchGuard - Detailed Workflow

## Overview

PatchGuard 通过五个内部阶段治理 AI 的代码修改行为：

1. **Analyze** - 分析问题，禁止改代码
2. **Lock** - 锁定计划，生成边界约束
3. **Implement** - 按约束执行修改
4. **Verify** - 检查边界合规性
5. **Promote** - Optional: 规则沉淀

**关键变化 (v3.0)**：
- 用户只输入 `/PatchGuard`，不需要手动触发阶段命令
- 内部阶段自动线性推进
- 只在关键确认点暂停等待用户确认

---

## Critical Rule: Explicit Invocation Override (v3.0)

### Highest Priority

**只要用户消息中显式出现 `/PatchGuard`，就必须进入 PatchGuard 流程。**

这是最高优先级规则，高于：
- 问题复杂度判断
- AI 自行判断
- 任何 shortcut 理由

### Forbidden Bypass Reasons

禁止使用以下理由绕过 PatchGuard：

| Forbidden Reason | Why Forbidden |
|------------------|---------------|
| "问题很简单" | 简单问题也需要边界确认 |
| "只改一两行" | 一行代码也可能引入 regression |
| "根因很明确" | 明确根因不等于边界明确 |
| "用户只是报告 bug" | 报告 bug 也需要分析 |
| "上一个 patch 已 PASS" | 新问题是新 patch |
| "可以快速修复" | 快速修复不等于安全修复 |
| "不需要完整流程" | 流程完整性是 PatchGuard 核心价值 |

### Required First Response

如果用户显式调用 `/PatchGuard`，AI 第一响应必须：

1. **进入 Analyze 阶段**
2. **禁止修改任何代码**
3. **确认或提议 Patch ID**
4. **处理未归档 patch（如存在）**

### Unarchived Patch Handling

如果检测到已有 patch session 未归档或未 ACCEPTED：

**AI 不得自行判断新消息属于旧 patch 或新 patch。**

必须询问用户：

```
检测到已有未归档 patch：
- <patch-id> (状态: <current_phase>)

请选择：
1. Continue existing patch
2. Reopen existing patch
3. Create new patch
4. Archive existing patch first
```

用户确认选择前：
- ❌ 不得修改任何代码
- ❌ 不得创建新 patch 目录
- ❌ 不得写入任何 patch 文件

---

## Activation Rule Check (v3.0)

### Activation Contract Priority

**Activation Contract 优先级最高。**

`/PatchGuard` 的激活规则写在 SKILL.md / Project Rules 中。

**不能依赖 PATCH_RULES.md 是否存在来决定是否激活。**

即使 PATCH_RULES.md 完全不存在，也必须激活。

### Activation Handshake

调用 `/PatchGuard` 后第一响应必须：

```
PatchGuard Activated
Phase: Analyze
Code Modification: Disabled
Patch ID: [Provided / Proposed / Pending Confirmation]
Existing Patch Session: [None / Found: <patch-id> (<state>)]

Rules Check:
- File: .rr/rules/PATCH_RULES.md
- Status: [Readable / Not Found / Error]
- Matching Promoted Rule: [Found: RR-XXX / Missing / Unknown]
- Candidate Needed: [Yes / No]

Next Step: Problem Understanding
```

### Rules Check Behavior

| PATCH_RULES.md Status | Behavior |
|-----------------------|----------|
| Readable, Rule Found | 引用 RR-XXX，继续 Analyze |
| Readable, Rule Missing | 标记 Missing，继续 Analyze，不阻塞 |
| Not Found | 标记 Unknown，继续 Analyze |
| Error | 标记 Unknown，继续 Analyze |

**缺少对应 rule 不阻塞 activation。**

### Missing Rule Does NOT Block

如果 PATCH_RULES.md 缺少对应 promoted rule：

- ✅ 继续进入 Analyze
- ❌ 不得自动写入 Promoted Rules
- ❌ 不得阻塞流程
- ✅ 在 Promote 阶段生成 RR Candidate
- ✅ 请求用户选择 Promote / Reject / Defer

### No Auto-Promotion

**即使发现需要新规则，AI 也不能自动写入 PATCH_RULES.md。**

AI 只能：
1. 检查是否存在 matching promoted rule
2. 在 Handshake 中报告检查结果
3. 如果缺失，标记 Candidate Needed: Yes
4. 在 Promote 阶段生成 RR Candidate
5. 等待用户确认 Promote / Reject / Defer

**Human 必须明确确认 Promote 后，规则才能写入 PATCH_RULES.md。**

---

## Pre-Lock Validation (v3.0)

### Purpose

**在 Analyze 进入 Lock 之前，必须做一致性检查。**

只有 Pre-Lock Validation 通过，才能进入 Lock。

如果失败，必须保持状态为 ANALYZE_DRAFT，列出 Blocking Questions。

---

### Seven Gates

#### Gate 1: Confirmation Gate

**Confirmation Needed 非空 → 禁止进入 Lock。**

| Rule | Enforcement |
|------|-------------|
| Confirmation Needed 非空 | CURRENT_STATE = ANALYZE_DRAFT |
| Confirmation Needed 非空 | 不生成 LOCKED_PLAN |
| Confirmation Needed 非空 | 不生成可执行 TASK_PACKET |

#### Gate 2: Scope Split Gate

**多个独立功能/风险域 → 必须询问是否拆分。**

Split Detection:
- 功能可独立交付
- 风险不同
- 依赖不同

用户确认前不得 Lock。

#### Gate 3: Boundary Consistency Gate

**TASK/PATCH_PLAN 文件必须出现在 Allowed Files。**

Forbidden 冲突 → 禁止 Lock。

#### Gate 4: Allowed Files Completeness Gate

**修改文件必须出现在 Allowed Files。**

不在 Allowed Files → 二选一：
- 加入 Allowed Files
- 移出当前 patch

#### Gate 5: P0/P1/P2 Separation Gate

**TASK_PACKET 只能包含 P0。**

P1/P2 混入 → 返回 Analyze 移除。

#### Gate 6: Dependency Decision Gate

**未确认外部依赖 → 禁止 Lock。**

用户确认依赖选择，或将功能移出。

#### Gate 7: Widget/File Name Consistency Gate

**新组件名必须有 Allowed Files 或复用已有文件。**

---

### Validation Result

| Result | Action |
|--------|--------|
| All Pass | 进入 Lock，生成 LOCKED 文件 |
| Any Fail | ANALYZE_DRAFT + Blocking Questions |

Blocking Questions Format:

```
Pre-Lock Validation Failed

Blocking Questions:
1. [Confirmation Needed]
2. [Scope Split]
3. [Boundary conflict]
4. [Allowed Files missing]
5. [P1/P2 in TASK_PACKET]
6. [Dependency unconfirmed]
7. [Widget/File inconsistent]

请逐一回答，确认后进入 Lock。
```

---

## Important: V3 Limitations

**V3-beta 是 Prompt Protocol，不是自动验证工具。**

- Verify 输出依赖 AI 自查，**仍然需要人工审查**
- **PASS 不代表业务完全正确**，只代表边界检查通过
- **WARNING 必须人工验证 Unverified Items 后，再决定是否提交**
- 如果存在 Unverified Items，状态不能是 PASS，只能是 WARNING 或 FAIL
- 未列入 Allowed 的新增文件一律 FAIL
- Implement 阶段禁止重新解释需求（检查 LAST_LOCKED_PLAN.md）

---

## User Entry Point

### Single Entry: /PatchGuard

用户只需要输入一个命令：

```
/PatchGuard

Patch ID: 2026-05-24-comment-replies

[需求或 bug 描述]
```

**不需要用户手动输入阶段命令。**

### Automatic Flow

```
/PatchGuard + 需求
↓
Analyze (自动)
↓
[暂停] 等待用户确认 Problem Understanding + Patch ID
↓
用户确认 "确认"
↓
Lock (自动)
↓
[暂停] 等待用户确认是否开始实现
↓
用户确认 "开始实现"
↓
Implement (自动)
↓
Verify (自动)
↓
[暂停] 输出 VERIFY_REPORT
↓
Optional: 询问 RR Candidate
↓
用户决定 Promote / Reject / Defer
↓
归档
```

### Pause Points

必须在以下确认点暂停等待用户：

| Pause Point | Wait For |
|-------------|----------|
| Analyze Result | Problem Understanding + Patch ID 确认 |
| Locked Plan | 是否开始实现确认 |
| Verify Result | 查看 VERIFY_REPORT，决定是否提交 |
| RR Candidate | Promote / Reject / Defer 决定 |

### User Commands at Pause Points

| Command | Meaning |
|---------|---------|
| `确认` | 确认当前阶段，继续下一阶段 |
| `修改分析` | 返回 Analyze，重新分析 |
| `缩小边界` | 返回 Analyze，缩小范围 |
| `修改 Patch ID` | 返回 Analyze，重新确认 ID |
| `开始实现` | 确认 Lock，进入 Implement |
| `停止` | 立即停止 |
| `回到 Analyze` | 返回 Phase 1 |
| `Promote` | 将 RR Candidate 写入 PATCH_RULES |
| `Reject` | 拒绝 RR Candidate |
| `Defer` | 暂缓 RR Candidate |

---

## Patch ID Rules

### User Can Explicitly Provide

```
/PatchGuard

Patch ID: 2026-05-24-comment-replies

[需求描述]
```

### AI Can Propose, But Must Confirm

如果用户没有提供 Patch ID：

```
Proposed Patch ID: comment-replies

请确认 Patch ID，或提供您自己的 Patch ID。
```

**必须等待用户确认。**

### Before Confirmation, Cannot Proceed

用户确认 Patch ID 前：
- ❌ 不得创建 `.rr/patches/<patch-id>/` 目录
- ❌ 不得写入任何 patch session 文件
- ❌ 不得进入 Lock 阶段

### After Locked, Cannot Modify

Patch ID 一旦进入 Locked Plan：
- ❌ 不得修改 Patch ID
- 如需修改，必须返回 Analyze 重新确认

### Format Rules

**只允许使用**：
- 小写字母 `a-z`
- 数字 `0-9`
- 短横线 `-`

**禁止使用**：
- 空格、中文、特殊符号、路径分隔符 `/`

**推荐格式**：
- `YYYY-MM-DD-short-topic`
- `issue-number-short-topic`
- `short-topic`

---

## Phase 1: Analyze

### Purpose

**禁止 AI 直接改代码。** 必须先进行结构化分析。

### Trigger

用户输入 `/PatchGuard` + 需求

### Rules

| Rule | Description |
|------|-------------|
| 禁止修改代码 | Analyze 阶段不允许修改任何文件 |
| 禁止生成 patch | 不输出代码片段或修改方案 |
| 禁止架构建议 | 不提出架构优化或重构建议 |
| 禁止扩大范围 | 不扩大问题范围 |
| 必须确认 Patch ID | 用户确认前不得创建目录 |

### Output: ANALYZE_REPORT.md

写入 `.rr/patches/<patch-id>/ANALYZE_REPORT.md`

| Section | Required | Priority | Description |
|---------|----------|----------|-------------|
| Problem Understanding | ✅ | **First** | 确认问题本身 |
| Patch ID | ✅ | Second | 用户确认或 AI 提议 |
| Problem Classification | ✅ | Third | 问题类型分类 |
| Impact Radius | ✅ | Fourth | 涉及和不涉及的文件/模块 |
| Regression Detection | ✅ | Fifth | 是否已知回归问题 |
| Risk Analysis | ✅ | Sixth | 可能误伤的模块 |
| Minimal Fix Strategy | ✅ | Seventh | 最小修改方案描述 |
| Version History | ✅ | Last | 多轮校对记录 |

### Hard Rule: Problem Understanding First

必须先填写 Problem Understanding：
- User Report
- Current Wrong Behavior
- Expected Behavior
- Confirmation Needed

**如果 Problem Understanding 未填写清楚，禁止进入 Problem Classification。**

### Hard Rule: Patch ID Confirmation

用户确认前状态必须是 **DRAFT**。
用户确认后状态才是 **LOCKED**。

### Escalate

如果问题无法在边界内解决：

```markdown
## Analysis Result: ESCALATE

当前问题无法在限定边界内局部解决。

原因: [具体说明]

建议: 升级为 Architecture Issue
```

**禁止**：自动扩大修改范围

---

## Phase 2: Lock

### Purpose

将分析结果结构化沉淀为边界约束和任务包。

### Trigger

- ANALYZE_REPORT.md 状态为 LOCKED
- Patch ID 已确认
- 用户输入 `确认`

### Output Files

写入 `.rr/patches/<patch-id>/`：

| File | Purpose |
|------|---------|
| BOUNDARY.md | Allowed/Forbidden Files/Behaviors/Stop Conditions |
| PATCH_PLAN.md | 本次 patch 的修改计划 |
| TASK_PACKET.md | 给 AI 执行的约束任务包 |
| CURRENT_STATE.md | 当前阶段状态 = locked |
| LAST_LOCKED_PLAN.md | 锁定的 Analyze 结果 |

### LOCKED_PLAN Rules

一旦锁定：

| Rule | Description |
|------|-------------|
| 禁止修改 Patch ID | Patch ID 不得变化 |
| 禁止重新解释需求 | 不允许在 Implement 阶段改变理解 |
| 禁止扩大 Allowed | 不允许增加新的 Allowed Files |
| 禁止减少 Forbidden | 不允许删除 Forbidden Files |
| 禁止新架构 | 不允许添加架构改进 |
| 禁止自由重构 | 不允许任何重构行为 |

---

## Phase 3: Implement

### Purpose

严格按 TASK_PACKET.md 约束执行修改。

### Trigger

用户输入 `开始实现`

### Input Source

**只能**来自 `.rr/patches/<patch-id>/TASK_PACKET.md`

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

### After Implement

修改完成后：
- 更新 `.rr/patches/<patch-id>/CURRENT_STATE.md` 状态为 `verify`
- 自动进入 Verify 阶段
- 不需要用户手动触发

---

## Phase 4: Verify

### Purpose

确认原始问题是否被修复，检查修改是否符合边界约束。

### Trigger

- Implement 完成
- 自动进入

### Output: VERIFY_REPORT.md

写入 `.rr/patches/<patch-id>/VERIFY_REPORT.md`

| Section | Required | Priority | Description |
|---------|----------|----------|-------------|
| Fix Verification | ✅ | **First** | 确认原始问题是否被修复 |
| Modified Files | ✅ | Second | 实际修改的文件列表 |
| New Files Created | ✅ | Third | 新增文件及其 Allowed 状态 |
| Boundary Check | ✅ | Fourth | Allowed/Forbidden 检查结果 |
| Unverified Items | ✅ | Fifth | 尚未验证的内容 |
| Evidence | ✅ | Sixth | 修改证据记录 |
| Manual Verification Required | ✅ | Seventh | 需人工验证的项目 |
| Risks | ✅ | Eighth | 风险项列表 |
| Final Recommendation | ✅ | Last | 提交建议 |

### Hard Rule: Fix Verification First

必须先填写 Fix Verification：
- Original Problem
- Expected Behavior
- Actual Behavior After Patch
- Verification Method
- Fix Result: Fixed / Not Fixed / Partially Fixed / Unknown

**如果 Fix Result 不是 Fixed，VERIFY_REPORT 不能是 PASS。**

### Status Definition

| Status | Condition | Action |
|--------|-----------|--------|
| PASS | Fix Result=Fixed、无越界、无 Forbidden、无未验证关键项 | ✅ 边界检查通过，可进入提交前人工审查 |
| WARNING | 无越界、无 Forbidden，但存在未验证项或 Fix Result=Partially Fixed | ⚠️ 必须人工验证后决定 |
| FAIL | Fix Result=Not Fixed/Unknown、触碰 Forbidden、超出 Allowed | ❌ 必须停止，根据风险决定回滚或返回 Analyze |

---

## Phase 5: Promote (Optional)

### Purpose

将可复用 regression pattern 沉淀为长期规则。

### Trigger

- Verify 完成
- 发现可复用 regression pattern
- AI 询问用户是否生成 RR Candidate

### Core Principle

**AI may propose RR Candidates, but only human-confirmed rules can be promoted to PATCH_RULES.**

### Process

```
Verify 完成 → AI 发现 regression pattern → 询问用户是否生成 RR Candidate
↓
用户决定：生成 / 不生成
↓
如果生成：AI 输出 RR Candidate
↓
用户决定：Promote / Reject / Defer
↓
如果 Promote：写入 .rr/rules/PATCH_RULES.md
```

### Promote Criteria

**Promotion requires**:
- Human explicitly confirms Promote

And **at least one** evidence criterion:
- Major Regression
- Typical Error Pattern
- Core Module confirmed by human
- Prevents scope creep or patch explosion
- Repeated error observed

---

## Directory Structure (v3.0)

### .rr/patches/<patch-id>/ (当前 patch)

- 每个 patch 独立目录
- 包含所有 patch 工作文件
- patch 结束后归档到 `.rr/archive/<patch-id>/`

### .rr/rules/ (全局长期)

- 永久存在
- 跨 patch 复用
- 只有重大 regression 才写入（promote rule）

### .rr/archive/<patch-id>/ (归档)

- 永久存在（可定期清理超过 30 天的归档）
- 存放已完成 patch 的历史文件

### No More Global current/ and state/

- ❌ 不再使用 `.rr/current/` 作为全局唯一当前任务目录
- ❌ 不再使用 `.rr/state/` 作为全局唯一状态目录
- ✅ 每个 patch 拥有独立目录 `.rr/patches/<patch-id>/`
- ✅ CURRENT_STATE.md 和 LAST_LOCKED_PLAN.md 都属于 patch 目录

---

## State Files

### CURRENT_STATE.md

记录当前 patch 的阶段状态。

写入 `.rr/patches/<patch-id>/CURRENT_STATE.md`

**Phase Flow**：
```
none → analyze (DRAFT) → analyze (LOCKED) → locked → implement → verify → done
```

### LAST_LOCKED_PLAN.md

保存锁定的 Analyze 结果。

写入 `.rr/patches/<patch-id>/LAST_LOCKED_PLAN.md`

**硬规则**：Implement 阶段禁止重新解释需求。

---

## Archive Process

### When to Archive

- Verify PASS 或 WARNING（已人工验证）
- 用户确认归档

### Steps

1. 用户确认归档
2. 移动 `.rr/patches/<patch-id>/` 到 `.rr/archive/<patch-id>/`
3. 可开始下一个 patch

---

## Emergency Stop

### Trigger

用户输入 `停止`

### Action

1. 立即停止当前操作
2. 回滚未提交的修改
3. 保留当前 patch 目录
4. 等待用户决定：回到 Analyze / 归档 / 继续其他工作

---

## Best Practices

### Analyze 阶段

1. 仔细检查 Impact Radius 是否正确
2. 确认 Regression Detection 是否准确
3. 验证 Risk Analysis 是否完整
4. **必须等待用户确认 Patch ID**

### Lock 阶段

1. 仔细检查 BOUNDARY.md 的 Allowed/Forbidden
2. 确认 Stop Conditions 是否覆盖关键风险
3. 验证 TASK_PACKET.md 是否清晰明确
4. **如需新增文件，必须提前列入 Allowed New Files**

### Implement 阶段

1. 只关注 TASK_PACKET.md 的内容
2. 随时关注 Stop Conditions
3. 有疑问立即停止，不要继续
4. **禁止新增未列入 Allowed 的文件**

### Verify 阶段

1. 认真检查 VERIFY_REPORT.md
2. **PASS 不代表业务完全正确**，仍需人工审查
3. **WARNING 必须人工验证后决定是否提交**
4. **FAIL 必须停止，根据风险决定回滚或返回 Analyze**

---

## Common Mistakes

| Mistake | Consequence | Prevention |
|---------|-------------|------------|
| 跳过 Analyze | 修改范围失控 | 坚持先分析后修改 |
| 单轮确认 | 边界不准确 | 多轮校对 |
| 忽略 Stop Conditions | 引入 regression | 立即停止并报告 |
| FAIL 后继续提交 | 破坏系统 | 必须停止 |

---

## Summary

| Principle | Description |
|-----------|-------------|
| Single Entry | 用户只输入 `/PatchGuard` |
| Automatic Progression | 内部阶段自动推进 |
| Pause at Confirmations | 只在关键点暂停 |
| One Patch One Directory | 每个 patch 独立 `.rr/patches/<patch-id>/` |
| Patch ID Must Confirm | AI 提议，Human 确认 |
| Locked Plan Cannot Change | Lock 后不得修改 Patch ID 或边界 |
| Problem First, Process Second | 流程服务于问题修复 |
| Human Controls Promote | AI 提议，Human 决定 |