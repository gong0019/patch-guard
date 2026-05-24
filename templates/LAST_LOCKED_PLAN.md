# LAST_LOCKED_PLAN.md

## Purpose

保存最近一次锁定的 Analyze 结果，防止 Implement 阶段重新解释需求。

---

## Hard Rule

**Implement 阶段禁止重新解释需求。**

如果 AI 在 Implement 时发现需要：
- 扩大修改范围
- 新增未 Allowed 文件
- 修改 Forbidden 文件
- 改变问题理解

**必须 STOP 并返回 rr analyze**，而不是擅自修改 LAST_LOCKED_PLAN。

---

## Content Structure

```markdown
# Last Locked Plan

## Lock Info

| Field | Value |
|-------|-------|
| locked_at | [timestamp] |
| locked_by | [user confirmation] |
| analyze_version | [v3 / v4 / ...] |

## Original Problem

[用户原始问题描述，从 ANALYZE_REPORT.md 复制]

## Locked Problem Classification

[问题分类，从 ANALYZE_REPORT.md 复制]

## Locked Impact Radius

涉及:
- [file/module list]

不涉及:
- [file/module list]

## Locked Minimal Fix Strategy

[最小修改方案，从 ANALYZE_REPORT.md 复制]

## Locked Allowed Files

[从 BOUNDARY.md 复制 Allowed Files]

## Locked Forbidden Files

[从 BOUNDARY.md 复制 Forbidden Files]

## Locked Stop Conditions

[从 BOUNDARY.md 复制 Stop Conditions]

## Change History

| Version | Change | Reason |
|---------|--------|--------|
| v1 | [change desc] | [reason] |

## Notes

[Any notes about this locked plan]
```

---

## When to Update

此文件只在以下情况更新：

1. **用户确认新计划**：用户输入 `确认计划` 后生成
2. **用户主动重新分析**：用户输入 `停止，重新分析` 后重新生成
3. **FAIL 后重新分析**：VERIFY_REPORT 状态为 FAIL，返回 analyze 后重新生成

**禁止 Implement 阶段擅自更新此文件。**

---

## Comparison Rule

Implement 阶段开始前，AI 必须对比：

| Current Understanding | LAST_LOCKED_PLAN | Match? |
|-----------------------|------------------|--------|
| [当前理解的问题] | [LOCKED 的原始问题] | [yes/no] |
| [当前理解的 Allowed] | [LOCKED 的 Allowed Files] | [yes/no] |
| [当前理解的 Forbidden] | [LOCKED 的 Forbidden Files] | [yes/no] |

如果不匹配，必须 STOP。

---

## Generated

- Generated: [timestamp]
- From: ANALYZE_REPORT.md [version]
- From: BOUNDARY.md [version]