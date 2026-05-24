# Archive Directory

## Purpose

存放已完成 patch 的归档文件，用于历史追溯和规则沉淀。

---

## Archive Structure (v3.0)

```
.rr/archive/
├── <patch-id>/                   # 使用 patch-id 作为目录名
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

## Archive Naming (v3.0)

使用 **patch-id** 作为归档目录名：

示例：
- `2024-05-24-comment-replies`
- `issue-123-fix-delete`
- `sample-comment-replies`

---

## Archive Process (v3.0)

```
patch 完成 → VERIFY_REPORT PASS/WARNING → 人工确认 → 归档
```

### Steps

1. 确认 VERIFY_REPORT 状态为 PASS 或 WARNING（已人工验证）
2. 用户确认归档
3. 移动 `.rr/patches/<patch-id>/` 到 `.rr/archive/<patch-id>/`
4. 可开始下一个 patch

**不再需要**：清空 `.rr/current/` 和 `.rr/state/`（v2.0 的做法）

---

## Archive Lifecycle (v3.0)

| Stage | Description |
|-------|-------------|
| Active | patch 进行中，文件在 `.rr/patches/<patch-id>/` |
| Archived | patch 完成，文件移至 `.rr/archive/<patch-id>/` |
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

- Patch ID: [patch-id]
- Date: [date]
- Status: [PASS / WARNING]

## Promote Criteria

| Criterion | Value | Notes |
|-----------|-------|-------|
| Human Confirmation | Required | 用户必须明确确认 Promote |
| Major Regression | [Yes / No] | [说明] |
| Typical Error Pattern | [Yes / No] | [说明] |
| Core Module | [Yes / No] | [说明] |

## Decision

- [ ] Promote to PATCH_RULES
- [ ] Reject
- [ ] Defer

## Reason

[具体说明]

## If Promote

- New RR ID: RR-XXX
- Rule Summary: [规则摘要]
```

---

## Notes

归档不是自动的，需要人工确认后执行。