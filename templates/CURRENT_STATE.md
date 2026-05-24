# CURRENT_STATE.md

## Purpose

记录当前 patch 的阶段状态，防止阶段跳跃和状态丢失。

写入 `.rr/patches/<patch-id>/CURRENT_STATE.md`

---

## State Fields

| Field | Value | Description |
|-------|-------|-------------|
| `patch_id` | [patch-id] | 当前 patch ID |
| `current_phase` | [none / analyze / locked / implement / verify / done] | 当前所处阶段 |
| `locked` | [true / false] | Analyze 结果是否已锁定 |
| `analyze_version` | [v1 / v2 / v3 / ...] | 当前 ANALYZE_REPORT 版本 |
| `boundary_version` | [v1 / v2 / ...] | 当前 BOUNDARY 版本 |
| `task_packet_version` | [v1 / v2 / ...] | 当前 TASK_PACKET 版本 |
| `last_modified` | [timestamp] | 最后更新时间 |

---

## Phase Flow (v3.0)

```
none → analyze (DRAFT) → analyze (LOCKED) → locked → implement → verify → done
```

### Valid Transitions

| From | To | Condition |
|------|-----|-----------|
| none | analyze | 用户输入 `/PatchGuard` |
| analyze (DRAFT) | analyze (LOCKED) | 用户确认 Patch ID + Problem Understanding |
| analyze (LOCKED) | locked | 用户输入 `确认` |
| locked | implement | 用户输入 `开始实现` |
| implement | verify | Implement 完成，自动进入 |
| verify | done | VERIFY_REPORT 状态为 PASS 或 WARNING（人工确认后） |
| any | analyze | 用户输入 `修改分析` 或 `回到 Analyze` |

### Invalid Transitions

| From | To | Reason |
|------|-----|--------|
| none | locked | 必须先完成 analyze |
| analyze (DRAFT) | implement | 必须先 LOCKED + locked |
| implement | locked | 禁止回退 |
| verify | implement | 禁止回退（除非 FAIL） |

---

## Current State Template

```markdown
# Current RR State

## Basic Info

| Field | Value |
|-------|-------|
| current_phase | [填写当前阶段] |
| locked | [true / false] |
| analyze_version | [版本号] |
| boundary_version | [版本号] |
| task_packet_version | [版本号] |
| last_modified | [timestamp] |

## Phase History

| Phase | Entered At | Status |
|-------|------------|--------|
| analyze | [timestamp] | [DRAFT / LOCKED / done] |
| commit | [timestamp] | [done / skipped] |
| implement | [timestamp] | [done / skipped] |
| verify | [timestamp] | [done / skipped] |

## Notes

[Any special notes about current state]
```

---

## Hard Rules

1. **禁止跳跃阶段**：必须按顺序 transition
2. **禁止 implement 阶段重新解释需求**：检查 LAST_LOCKED_PLAN.md
3. **每次阶段变化必须更新此文件**
4. **FAIL 状态必须停止，并根据风险决定回滚或返回 rr analyze**

---

## Generated

- Generated: [timestamp]
- Project: [project name]