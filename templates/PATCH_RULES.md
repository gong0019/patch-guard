# PATCH_RULES.md

## Purpose

长期 Regression Rules，跨 patch 复用，防止 AI 重复犯错。

---

## Important: Not Every Bug Becomes a Rule

**不是每个 bug 都写入 PATCH_RULES。**

只有符合以下条件才写入：
1. **重大 regression**：导致系统崩溃、数据丢失、核心功能失效
2. **典型错误模式**：其他 AI 可能重犯的常见错误
3. **核心模块**：修改风险高，需要特别提醒

**禁止**：将一次性 bug、低风险修复、简单 typo 写入长期规则。

---

## Promote Rule Process

```
bug fixed → VERIFY_REPORT 完成 → 评估是否 promote → 用户确认 → 写入 PATCH_RULES
```

### Promote Criteria

| Criterion | Required | Description |
|-----------|----------|-------------|
| Severity | High / Critical | 是否影响核心功能 |
| Pattern | Yes | 是否为典型错误模式 |
| Recurrence Risk | High | 其他 AI 是否可能重犯 |
| User Confirmation | Required | 用户必须明确确认 |

### Promote Decision Template

```markdown
## Promote Decision

| Criterion | Value | Notes |
|-----------|-------|-------|
| Severity | [High / Medium / Low] | [具体说明] |
| Pattern | [Yes / No] | [是否典型模式] |
| Recurrence Risk | [High / Medium / Low] | [复现风险评估] |
| Decision | [Promote / Not Promote] | [最终决定] |
| Reason | [说明] | [为什么 promote 或不 promote] |
```

---

## Rule Format

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
- [禁止行为 2]

### Must Do

涉及 [功能] 时，必须：
- [必须行为 1]

### Reason

[历史 regression 描述，为什么需要这个规则]

### Severity

[Critical / High / Medium]

### Related Bugs

[关联的历史 bug ID 或描述]
```

---

## Rule Index

| RR | Scope | Severity | Summary |
|----|-------|----------|---------|
| RR-001 | [module] | [level] | [rule summary] |

---

## Rule Lifecycle

| Stage | Description |
|-------|-------------|
| Created | 用户确认 promote 后创建 |
| Active | 所有 patch 必须检查相关 RR |
| Deprecated | 规则不再适用，标记 deprecated |
| Removed | 确认无用后删除 |

---

## Deprecated Rules

已废弃规则保留在此，标记 `[DEPRECATED]`：

```markdown
## RR-XXX [DEPRECATED]

Deprecated at: [timestamp]
Reason: [废弃原因]
```

---

## Notes

- 规则数量保持在精简范围（建议 < 20 条）
- 规则必须有实际防止 regression 的价值
- 定期 review 规则有效性

---

## Generated

- Project: [project name]
- Last Updated: [timestamp]