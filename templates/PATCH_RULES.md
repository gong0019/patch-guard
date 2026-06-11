# PATCH_RULES.md

## Purpose

长期 Regression Rules，跨 patch 复用，防止 AI 重复犯错。

---

## Core Principle

**AI may propose RR Candidates, but only human-confirmed rules can be promoted to PATCH_RULES.**

PATCH_RULES.md should contain only active, human-confirmed regression rules.

**禁止 AI 自行 promote**。AI 只能生成 RR Candidate，最终决策权在 Human。

---

## Activation Rule Check

### Rules File Does NOT Block Activation

**PATCH_RULES.md 是否存在不影响 /patch-guard 激活。**

即使 PATCH_RULES.md 完全不存在、无法读取、或缺少对应规则：

- `/patch-guard` 仍然必须激活
- 必须进入 Analyze 阶段
- 不能跳过流程

### No Auto-Promotion Even for Activation Rules

**即使是 activation-related rule，也必须遵守 Promote Governance。**

AI 在发现缺失规则时：
- ✅ 可以在 Activation Handshake 标记 Candidate Needed: Yes
- ✅ 可以在 Promote 阶段生成 RR Candidate
- ❌ 不能自动写入 PATCH_RULES.md
- ❌ 不能跳过 Human 确认

**Human 必须明确确认 Promote 后，规则才能写入。**

---

## Important: Not Every Bug Becomes a Rule

**不是每个 bug 都写入 PATCH_RULES。**

AI may use the following criteria to propose an RR Candidate.
These criteria are evidence for promotion, not automatic promotion rules.
Only human-confirmed candidates can become active PATCH_RULES.

**禁止**：将一次性 bug、低风险修复、简单 typo 写入长期规则。

---

## Definition: Major Regression

Only count as major regression if it causes:

- Data loss
- Incorrect database write
- Broken save pipeline
- Broken render pipeline for a core page
- Authentication / permission / payment / order flow failure
- Irreversible state corruption
- Production-blocking error

**不包括**：
- UI 显示小问题
- 非核心页面样式问题
- 文案错误
- 一次性配置错误

---

## Definition: Typical Error Pattern

Typical AI patch mistakes include:

- Template changed but JS state not updated
- Update path fixed but insert path missed
- Render fixed but save pipeline missed
- Create/copy mode missed
- Local bug fixed by unrelated refactor
- Patch scope expanded beyond locked boundary
- Verification claims fixed but no evidence was provided
- **Skipping Analyze when user explicitly invoked /patch-guard** (CRITICAL)
- **Bypassing flow with "simple problem" or "quick fix" excuses** (CRITICAL)

---

## Critical Rule: Explicit Invocation

**When user explicitly invokes `/patch-guard`, never skip Analyze.**

Typical violation excuses:
- "root cause is clear"
- "only two lines need changing"
- "I can quickly fix this"
- "previous patch was already PASS"
- "this is a simple bug"

Required behavior:
- Always enter Analyze first
- Output minimum ANALYZE_REPORT with Problem Understanding + Patch ID + Boundary
- Wait for user confirmation before any code modification

---

## Definition: Core Module

AI must not assume a module is core.

A module is core only if:
- Listed in `.rr/rules/CORE_MODULES.md`
- Or explicitly confirmed by human during RR Analyze / RR Commit

If `CORE_MODULES.md` does not exist, AI may suggest a core module candidate, but cannot use "core module" as promotion evidence without human confirmation.

---

## Do Not Promote

Do not promote if:

- The bug is one-off
- The rule is vague
- The rule only describes this exact patch
- The rule cannot guide future patches
- The rule duplicates an existing rule
- There is no human confirmation
- The only evidence is AI speculation

---

## RR Candidate Template

每个 RR Candidate 必须包含：

```markdown
## RR Candidate: C-XXX

### Source Patch

- Patch ID: [YYYY-MM-DD-patch-XXX]
- Date: [date]

### Original Problem

[原始问题描述]

### Proposed Rule

修改 [module/file] 时：

必须同步检查：
- [相关模块 1]
- [相关模块 2]

禁止：
- [禁止行为 1]

### Evidence

[具体 regression 描述]

### Promotion Criteria Matched

| Criterion | Matched? | Evidence |
|-----------|----------|----------|
| Major Regression? | Yes/No | [说明] |
| Typical Error Pattern? | Yes/No | [说明] |
| Core Module? | Yes/No | [说明] |

### Duplicate Check

- Existing RR Rules: [列出可能重复的规则]
- Is Duplicate? | Yes/No |

### Human Decision

| Decision | Promote / Reject / Defer |
|----------|--------------------------|
| Decision By | [human name] |
| Decision Date | [date] |
| Decision Reason | [说明] |

**Status**: [Candidate / Promoted / Rejected / Deferred]

If Promoted, assigned RR ID: RR-XXX
```

---

## Promotion Checklist

**Mandatory**:
| Criterion | Yes/No | Evidence |
|-----------|--------|----------|
| Human Decision = Promote? | **Required** | [human name, date] |

**Evidence (at least one should match)**:
| Criterion | Yes/No | Evidence |
|-----------|--------|----------|
| Has this error happened before? | | |
| Did it cause data loss, crash, or core feature failure? | | |
| Is it likely to recur in similar patches? | | |
| Is the affected module explicitly marked as core? | | |
| Would this rule prevent future scope creep or patch explosion? | | |
| Is the rule specific and actionable? | | |

**Blockers (any Yes = do not promote)**:
| Criterion | Yes/No | Evidence |
|-----------|--------|----------|
| Does it duplicate an existing rule? | | |
| Is it too narrow or one-off? | | |

**Key Rule**: If Human Decision ≠ Promote, even with strong evidence, candidate cannot become active rule. |

---

## Promote Rule Process

```
bug fixed → VERIFY_REPORT 完成 → AI 生成 RR Candidate → Human 决策 → Promote / Reject / Defer
```

**关键规则**：
- AI 只能生成 RR Candidate
- AI 不能自行 promote
- 只有 Human 明确确认 Promote 后，规则才能进入长期 PATCH_RULES
- Reject / Defer 不得成为 active rule

---

## Promoted Rules Format

```markdown
## RR-XXX

### Scope

[规则适用的模块/文件范围]

### Must Check

修改 [module/file] 时，必须同步检查：
- [相关模块 1]
- [相关模块 2]

### Forbidden

涉及 [功能] 时，禁止：
- [禁止行为 1]

### Reason

[历史 regression 描述，为什么需要这个规则]

### Severity

[Critical / High / Medium]

### Promoted By

[Human name] on [date]

### Related Bugs

[关联的历史 bug ID 或描述]
```

---

## Rule Index

### Promoted Rules

| RR | Scope | Severity | Summary | Promoted By |
|----|-------|----------|---------|-------------|
| RR-001 | [module] | [level] | [rule summary] | [human] |

### RR Candidates (Not Promoted)

| Candidate | Status | Proposed Rule | Human Decision |
|-----------|--------|---------------|----------------|
| C-XXX | Candidate | [rule summary] | Pending |

---

## Rule Lifecycle

| Stage | Description |
|-------|-------------|
| Candidate | AI 提出，等待 Human 决策 |
| Promoted | Human 确认，成为 active rule |
| Rejected | Human 拒绝，不成为 active rule |
| Deferred | Human 暂缓，待后续评估 |
| Deprecated | 规则不再适用，标记 deprecated |
| Removed | 确认无用后删除 |

---

## Notes

- 规则数量保持在精简范围（建议 < 20 条）
- 规则必须有实际防止 regression 的价值
- 定期 review 规则有效性
- **AI 不能自行 promote**

---

## Generated

- Project: [project name]
- Last Updated: [timestamp]