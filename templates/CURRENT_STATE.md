# CURRENT_STATE.md

## Purpose

记录当前 patch 的阶段状态，防止阶段跳跃和状态丢失。

写入 `.rr/patches/<patch-id>/CURRENT_STATE.md`

---

## State Fields

| Field | Value | Description |
|-------|-------|-------------|
| `patch_id` | [patch-id] | 当前 patch ID |
| `current_phase` | [none / analyze / pre_lock / locked / implement / verify / done] | 当前所处阶段 |
| `locked` | [true / false] | Analyze 结果是否已锁定 |
| `validation_passed` | [true / false / pending] | Pre-Lock Validation 是否通过 |
| `blocking_questions` | [list / empty] | 未确认的问题列表 |
| `analyze_version` | [v1 / v2 / v3 / ...] | 当前 ANALYZE_REPORT 版本 |
| `boundary_version` | [v1 / v2 / ...] | 当前 BOUNDARY 版本 |
| `task_packet_version` | [v1 / v2 / ...] | 当前 TASK_PACKET 版本 |
| `last_modified` | [timestamp] | 最后更新时间 |

---

## Phase Flow (v3.0)

```
none → analyze (DRAFT) → pre_lock → analyze (LOCKED) → locked → implement → verify → done
```

### Valid Transitions

| From | To | Condition |
|------|-----|-----------|
| none | analyze | 用户输入 `/PatchGuard` |
| analyze (DRAFT) | pre_lock | 用户确认 Patch ID + Problem Understanding |
| pre_lock | analyze (LOCKED) | Pre-Lock Validation 全部通过 |
| pre_lock | analyze (DRAFT) | Pre-Lock Validation 失败 + Blocking Questions |
| analyze (LOCKED) | locked | 用户输入 `确认` + Validation 已通过 |
| locked | implement | 用户输入 `开始实现` |
| implement | verify | Implement 完成，自动进入 |
| verify | done | VERIFY_REPORT 状态为 PASS 或 WARNING（人工确认后） |
| any | analyze | 用户输入 `修改分析` 或 `回到 Analyze` |

### Invalid Transitions

| From | To | Reason |
|------|-----|--------|
| none | locked | 必须先完成 analyze + pre_lock validation |
| analyze (DRAFT) | locked | **必须先 Pre-Lock Validation 通过** |
| analyze (DRAFT) | implement | 必须先 LOCKED + locked |
| pre_lock | locked | **必须 Validation 通过** |
| implement | locked | 禁止回退 |
| verify | implement | 禁止回退（除非 FAIL） |

---

## Locked Phase Conditions (v3.0)

**只有当以下条件全部满足时，才能写 current_phase = locked：**

| Condition | Check |
|-----------|-------|
| Confirmation Needed 为空 | ANALYZE_REPORT Confirmation Needed = N/A 或 empty |
| Patch ID 已确认 | 用户明确确认 |
| Pre-Lock Validation 通过 | validation_passed = true |
| Boundary 无冲突 | TASK_PACKET 文件全部在 Allowed Files |
| P0/P1/P2 已分离 | TASK_PACKET 只包含 P0 |
| 外部依赖已确认 | 或已移出当前 patch |
| 用户明确确认 Lock | 用户输入 `确认` |

**否则必须写：**

```
current_phase: analyze (DRAFT)
validation_passed: false
blocking_questions: [list of questions]
```

---

## Current State Template

```markdown
# Current State

## Patch ID

| Field | Value |
|-------|-------|
| patch_id | [patch-id] |
| current_phase | [analyze / pre_lock / locked / implement / verify / done] |
| locked | [true / false] |
| validation_passed | [true / false / pending] |
| last_modified | [timestamp] |

## Pre-Lock Validation Status

| Gate | Status | Notes |
|------|--------|-------|
| Confirmation Gate | [pass / fail] | [notes] |
| Scope Split Gate | [pass / fail / skip] | [notes] |
| Boundary Consistency Gate | [pass / fail] | [notes] |
| Allowed Files Completeness Gate | [pass / fail] | [notes] |
| P0/P1/P2 Separation Gate | [pass / fail] | [notes] |
| Dependency Decision Gate | [pass / fail / skip] | [notes] |
| Widget/File Name Consistency Gate | [pass / fail] | [notes] |

## Blocking Questions (if validation_passed = false)

1. [question 1]
2. [question 2]
...

## Phase History

| Phase | Entered At | Status |
|-------|------------|--------|
| analyze | [timestamp] | [DRAFT / LOCKED / done] |
| pre_lock | [timestamp] | [passed / failed] |
| locked | [timestamp] | [done / pending] |
| implement | [timestamp] | [done / skipped] |
| verify | [timestamp] | [done / skipped] |

## Notes

[Any special notes about current state]
```

---

## Hard Rules (v3.0)

1. **禁止跳跃阶段**：必须按顺序 transition
2. **禁止跳过 Pre-Lock Validation**：必须通过所有 Gate
3. **禁止 Confirmation Needed 非空时进入 Lock**
4. **禁止 implement 阶段重新解释需求**：检查 LAST_LOCKED_PLAN.md
5. **每次阶段变化必须更新此文件**
6. **FAIL 状态必须停止**，根据风险决定回滚或返回 analyze

---

## Generated

- Generated: [timestamp]
- Project: [project name]