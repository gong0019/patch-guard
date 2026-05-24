# PatchGuard - Quick Start Guide

## What is PatchGuard?

PatchGuard 是一个 **AI Patch Governance System**，用于治理 AI Coding 工具的代码修改行为。

**核心目标**：让 AI 在修改代码时：
- 不乱改
- 不扩大问题范围
- 不破坏已有系统
- 不发生上下文漂移
- 不发生架构侵蚀

---

## Important: V3 Limitations

**V3-beta 是 Prompt Protocol，不是自动验证工具。**

- Verify 输出依赖 AI 自查，**仍然需要人工审查**
- **PASS 不代表业务完全正确**，只代表边界检查通过
- **WARNING 必须人工验证 Unverified Items 后，再决定是否提交**
- Unverified Items 存在时，状态不能是 PASS

### Problem-Focused Governance

**Analyze 阶段第一目标不是分类，而是确认问题本身**：
- 当前错误行为是什么
- 期望行为是什么
- 用户是否确认

**Verify 阶段第一目标不是边界检查，而是确认原始问题是否被修复**。

如果问题未修复，即使边界合规，patch 也是失败的。

---

## Status Definition

| Status | Condition | Meaning |
|--------|-----------|---------|
| PASS | 无越界、无 Forbidden、无未验证关键项 | 边界检查通过，可进入提交前人工审查 |
| WARNING | 无越界、无 Forbidden，但存在未验证项 | **必须人工验证 Unverified Items 后决定** |
| FAIL | 触碰 Forbidden、超出 Allowed、违反 Locked Plan | 必须停止，根据风险决定回滚或返回 Analyze |

---

## Critical Rule: Explicit Invocation (v3.0)

**只要用户显式调用 `/PatchGuard`，就必须进入流程。**

### Forbidden Shortcut Excuses

禁止使用以下理由绕过 PatchGuard：

| Forbidden Excuse | Why Forbidden |
|------------------|---------------|
| "问题很简单" | 简单问题也需要边界确认 |
| "只改一两行" | 一行代码也可能引入 regression |
| "根因很明确" | 明确根因不等于边界明确 |
| "可以快速修复" | 快速修复不等于安全修复 |
| "上一个 patch 已 PASS" | 新问题是新 patch |

### Required Behavior

如果用户显式调用 `/PatchGuard`：

1. **必须进入 Analyze**
2. **禁止直接修改代码**
3. **输出最小 ANALYZE_REPORT**
4. **等待用户确认**

### Unarchived Patch Handling

如果检测到已有未归档 patch：

```
检测到已有未归档 patch：
- <patch-id> (状态: <current_phase>)

请选择：
1. Continue existing patch
2. Reopen existing patch
3. Create new patch
4. Archive existing patch first
```

**在用户确认前，不得修改代码。**

---

## Activation Rule Check (v3.0)

### Activation Handshake

调用 `/PatchGuard` 后第一响应：

```
PatchGuard Activated
Phase: Analyze
Code Modification: Disabled
Patch ID: [Provided / Proposed / Pending]

Rules Check:
- Matching Promoted Rule: [Found: RR-XXX / Missing / Unknown]
- Candidate Needed: [Yes / No]

Next Step: Problem Understanding
```

### Rules Check Does NOT Block

**缺少 promoted rule 不阻塞 activation。**

即使 PATCH_RULES.md 不存在，也必须进入 Analyze。

| Rules Status | Action |
|--------------|--------|
| Found | 引用规则，继续 Analyze |
| Missing | 标记缺失，继续 Analyze |
| Unknown | 标记未知，继续 Analyze |

### No Auto-Promotion

**AI 不能自动写入 PATCH_RULES.md。**

即使发现需要新规则：
- 只能生成 RR Candidate
- Human 必须确认 Promote
- 未确认不能写入

---

## Pre-Lock Validation (v3.0)

### Purpose

**Analyze → Lock 之前必须做一致性检查。**

只有 Validation 通过，才能进入 Lock。

### Seven Gates

| Gate | Rule |
|------|------|
| Confirmation Gate | Confirmation Needed 非空 → 禁止 Lock |
| Scope Split Gate | 多个独立功能 → 询问是否拆分 |
| Boundary Consistency Gate | TASK 文件必须在 Allowed Files |
| Allowed Files Completeness Gate | 修改文件必须列入 Allowed |
| P0/P1/P2 Separation Gate | TASK_PACKET 只能包含 P0 |
| Dependency Decision Gate | 未确认依赖 → 禁止 Lock |
| Widget/File Name Gate | 新组件名必须有 Allowed 或复用 |

### Validation Failed

如果任何 Gate 失败：

```
Pre-Lock Validation Failed

Blocking Questions:
1. [question]
2. [question]

请逐一回答，确认后进入 Lock。
```

**Blocking Questions 非空时，Status 必须保持 DRAFT。**

---

## Quick Start

### Single Entry Point: /PatchGuard

**用户只需要输入一个命令，后面跟需求或 bug 描述。**

```
/PatchGuard

Patch ID: 2026-05-24-comment-replies

[需求或 bug 描述]
```

AI 会自动：
1. 进入 Analyze 阶段
2. 创建 `.rr/patches/<patch-id>/` 目录
3. 根据 templates/ 生成必要的文件
4. 在确认点暂停等待用户确认

**不需要手动触发阶段命令。**

---

## Automatic Flow

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
Implement (自动) → Verify (自动)
↓
[暂停] 输出 VERIFY_REPORT
↓
Optional: RR Candidate → 用户决定
↓
归档
```

---

## User Commands at Pause Points

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

### If User Provides Patch ID

```
/PatchGuard

Patch ID: 2026-05-24-comment-replies

[需求描述]
```

AI 直接使用用户提供的 Patch ID。

### If User Does Not Provide Patch ID

```
/PatchGuard

[需求描述]
```

AI 会提出：

```
Proposed Patch ID: comment-replies

请确认 Patch ID，或提供您自己的 Patch ID。
```

**必须等待用户确认。**

### Patch ID Format

**只允许使用**：
- 小写字母 `a-z`
- 数字 `0-9`
- 短横线 `-`

**禁止使用**：
- 空格、中文、特殊符号、路径分隔符

**推荐格式**：
- `YYYY-MM-DD-short-topic`
- `issue-number-short-topic`
- `short-topic`

---

## Directory Structure (v3.0)

每个 patch 拥有独立目录：

```
project/
├── .rr/
│   ├── patches/
│   │   ├── <patch-id>/
│   │   │   ├── ANALYZE_REPORT.md
│   │   │   ├── BOUNDARY.md
│   │   │   ├── PATCH_PLAN.md
│   │   │   ├── TASK_PACKET.md
│   │   │   ├── VERIFY_REPORT.md
│   │   │   ├── CURRENT_STATE.md
│   │   │   └── LAST_LOCKED_PLAN.md
│   │   │
│   │   └── <another-patch-id>/    # 多 patch 并存
│   │       └── ...
│   │
│   ├── rules/
│   │   └── PATCH_RULES.md         # 全局长期规则
│   │
│   └── archive/
│       └── <patch-id>/            # 已完成 patch 归档
```

### Key Changes from v2.0

| v2.0 | v3.0 |
|------|------|
| `.rr/current/` 全局唯一 | `.rr/patches/<patch-id>/` 每个 patch 独立 |
| `.rr/state/` 全局唯一 | 状态文件移入 patch 目录 |
| 单 patch session | 多 patch session 并存 |

---

## Best Practices

1. **始终先分析**：不要跳过 Analyze 阶段
2. **确认 Patch ID**：用户确认前 AI 不能创建目录
3. **多轮校对**：仔细检查分析结果，确保边界正确
4. **关注 Stop Conditions**：AI 报告停止时，认真对待
5. **WARNING 必须人工验证**：存在未验证项时必须人工确认
6. **PASS 仍需审查**：边界检查通过不代表业务完全正确
7. **新增文件需提前规划**：在 Analyze 阶段列入 Allowed New Files

---

## Example Session

### Start

```
/PatchGuard

Patch ID: 2026-05-24-fix-comment-delete

删除评论时，关联的追评数据没有被清理，导致数据残留。
```

### Analyze Output (DRAFT)

AI 输出 ANALYZE_REPORT.md，包含：
- Problem Understanding
- Patch ID: 2026-05-24-fix-comment-delete
- Impact Radius
- Minimal Fix Strategy

### User Confirm

```
确认
```

### Lock (Automatic)

AI 生成：
- BOUNDARY.md
- PATCH_PLAN.md
- TASK_PACKET.md

### User Confirm to Implement

```
开始实现
```

### Implement + Verify (Automatic)

AI 修改代码，自动进入 Verify，输出 VERIFY_REPORT.md。

### If Regression Pattern Found

AI 询问：

```
发现可复用 regression pattern：删除评论时需要同步清理追评数据。
是否生成 RR Candidate？
```

用户决定：

```
Promote
```

### Archive

```
归档
```

---

## More Examples

查看 `.rr-example/` 目录获取完整工作流示例。