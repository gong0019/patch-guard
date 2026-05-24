# CURRENT_STATE.md

## Purpose

记录当前 RR 流程的阶段状态，防止阶段跳跃和状态丢失。

---

## State Fields

| Field | Value | Description |
|-------|-------|-------------|
| `current_phase` | [none / analyze / commit / implement / verify / done] | 当前所处阶段 |
| `locked` | [true / false] | Analyze 结果是否已锁定 |
| `analyze_version` | [v1 / v2 / v3 / ...] | 当前 ANALYZE_REPORT 版本 |
| `boundary_version` | [v1 / v2 / ...] | 当前 BOUNDARY 版本 |
| `task_packet_version` | [v1 / v2 / ...] | 当前 TASK_PACKET 版本 |
| `last_modified` | [timestamp] | 最后更新时间 |

---

## Phase Flow

```
none → analyze (DRAFT) → analyze (LOCKED) → commit → implement → verify → done
```

### Valid Transitions

| From | To | Condition |
|------|-----|-----------|
| none | analyze | 用户触发 `rr analyze` |
| analyze (DRAFT) | analyze (LOCKED) | 用户输入 `确认计划` |
| analyze (LOCKED) | commit | 状态已 locked |
| commit | implement | 用户触发 `开始执行` |
| implement | verify | 用户触发 `rr verify` |
| verify | done | VERIFY_REPORT 状态为 PASS 或 WARNING（人工确认后） |
| any | analyze | 用户输入 `停止，重新分析` |

### Invalid Transitions

| From | To | Reason |
|------|-----|--------|
| none | commit | 必须先完成 analyze |
| analyze (DRAFT) | implement | 必须先 LOCKED |
| implement | commit | 禁止回退到 commit |
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