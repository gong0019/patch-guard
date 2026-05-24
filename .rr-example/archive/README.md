# Archive Directory

## Purpose

存放已完成 patch 的归档文件，用于历史追溯和规则沉淀。

---

## Archive Structure

```
.rr/archive/
├── YYYY-MM-DD-patch-XXX/
│   ├── ANALYZE_REPORT.md
│   ├── BOUNDARY.md
│   ├── PATCH_PLAN.md
│   ├── TASK_PACKET.md
│   ├── VERIFY_REPORT.md
│   ├── CURRENT_STATE.md (final)
│   └── LAST_LOCKED_PLAN.md
│   └── promote-decision.md (if applicable)
```

---

## Archive Naming

格式：`YYYY-MM-DD-patch-XXX`

- `YYYY-MM-DD`: patch 完成日期
- `patch-XXX`: patch 序号或简短描述

示例：
- `2024-05-24-variants-echo-fix`
- `2024-05-24-patch-001`

---

## Archive Process

```
patch 完成 → VERIFY_REPORT PASS/WARNING → 人工确认 → 归档
```

### Steps

1. 确认 VERIFY_REPORT 状态为 PASS 或 WARNING（已人工验证）
2. 创建归档目录：`.rr/archive/YYYY-MM-DD-patch-XXX/`
3. 移动 `.rr/current/` 文件到归档目录
4. 移动 `.rr/state/` 文件到归档目录
5. 清空 `.rr/current/` 和 `.rr/state/`（准备下一个 patch）
6. 评估是否 promote rule（写入 PATCH_RULES）

---

## Archive Lifecycle

| Stage | Description |
|-------|-------------|
| Active | patch 进行中，文件在 `.rr/current/` |
| Archived | patch 完成，文件移至 `.rr/archive/` |
| Promoted | 如果符合条件，规则写入 `.rr/rules/PATCH_RULES.md` |

---

## Retention Policy

- 保留最近 30 天的归档
- 超过 30 天的归档可删除（或移至长期存储）
- 已 promote 的规则永久保留

---

## Promote Decision

归档时评估是否写入长期规则：

```markdown
# Promote Decision

## Patch Info

- Patch: [patch name]
- Date: [date]
- Status: [PASS / WARNING]

## Promote Criteria

| Criterion | Value | Notes |
|-----------|-------|-------|
| Severity | [High / Medium / Low] | [说明] |
| Pattern | [Yes / No] | [说明] |
| Recurrence Risk | [High / Medium / Low] | [说明] |

## Decision

- [ ] Promote to PATCH_RULES
- [ ] Not Promote

## Reason

[具体说明]

## If Promote

- New RR ID: RR-XXX
- Rule Summary: [规则摘要]
```

---

## Notes

归档不是自动的，需要人工确认后执行。