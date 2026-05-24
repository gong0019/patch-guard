# ANALYZE_REPORT.md

## Status
[DRAFT / LOCKED]

---

## Patch ID (v3.0)

| Field | Value |
|-------|-------|
| Patch ID | [patch-id] |
| Source | User provided / AI proposed |
| Confirmed | [Yes / No - AI proposed must wait for user confirmation] |

**如果 AI proposed 且用户未确认，不得创建 .rr/patches/<patch-id>/ 目录。**

---

## Problem Understanding (v3.0)

**Analyze 阶段必须先确认问题本身，再做分类和边界。**

### User Report

[用户原始问题描述，直接引用用户的话]

### Current Wrong Behavior

[当前系统的错误行为是什么？具体表现？]

### Expected Behavior

[用户期望的正确行为是什么？]

### Confirmation Needed

[需要与用户确认的问题点，如有]

**Confirmation Needed 规则：**
- 如果 Confirmation Needed 非空，Status 必须是 DRAFT
- Confirmation Needed 非空时，禁止进入 Lock
- Confirmation Needed 非空时，禁止生成 TASK_PACKET

示例 Confirmation Needed：
- 是否拆分 patch（activity-timeline vs global-quick-capture）
- 外部依赖选择（hotkey_manager vs macos_global_shortcut）
- Widget 命名选择（ActivityTimelineWidget vs ActivityTimelineScreen）
- 是否包含 AI activity

**如果 User Report / Current Wrong Behavior / Expected Behavior 未填写清楚，禁止进入 Problem Classification。**

---

## Pre-Lock Validation Status (v3.0)

### Gate Check Results

| Gate | Status | Notes |
|------|--------|-------|
| Confirmation Gate | [pass / fail] | Confirmation Needed = [empty / has items] |
| Scope Split Gate | [pass / fail / skip] | [single scope / split detected / not applicable] |
| Boundary Consistency Gate | [pending] | 待 BOUNDARY.md 生成后检查 |
| Allowed Files Completeness Gate | [pending] | 待 BOUNDARY.md 生成后检查 |
| P0/P1/P2 Separation Gate | [pending] | 待 TASK_PACKET.md 生成后检查 |
| Dependency Decision Gate | [pass / fail / skip] | [confirmed / unconfirmed / not applicable] |
| Widget/File Name Consistency Gate | [pass / fail] | [consistent / inconsistent] |

### Blocking Questions (if any gate = fail)

如果任何 Gate 为 fail，列出 Blocking Questions：

1. [question from Confirmation Needed]
2. [question from Scope Split]
3. [question from Dependency]
4. [question from Widget/File Name]
...

**Blocking Questions 非空时，Status 必须保持 DRAFT。**

---

## Problem

选择一项：

- [ ] UI Bug
- [ ] State Bug
- [ ] Logic Bug
- [ ] Save Pipeline Bug
- [ ] Render Bug
- [ ] Workflow Bug
- [ ] Architecture Bug
- [ ] Regression

分类结果: [填写分类]

---

## Impact Radius

### 涉及

- [file/module 1]
- [file/module 2]
- [file/module 3]

### 不涉及

- [file/module 1]
- [file/module 2]
- [file/module 3]

---

## Regression Detection

选择一项：

- [ ] New Issue - 新发现的问题
- [ ] Known Regression - 已知的回归问题
- [ ] Requirement Change - 需求变更导致
- [ ] Architecture Defect - 架构缺陷

检测结果: [填写检测结果]

---

## Risk Analysis

### 可能误伤

- [风险项 1]
- [风险项 2]
- [风险项 3]

### 关联已有 RR

- [RR-XXX]
- [RR-YYY]
- 或: 无

---

## Minimal Fix Strategy

[描述最小修改方案]

**禁止**:
- 大范围重构
- 顺手优化
- 架构改动

**允许**:
- 最小化修改
- 针对性修复
- 保持现有结构

---

## Version History

| Version | Time | Change |
|---------|------|--------|
| v1 | [timestamp] | [initial analysis] |
| v2 | [timestamp] | [user correction: ...] |
| v3 | [timestamp] | [user补充: ...] |
| LOCKED | [timestamp] | [用户确认锁定] |

---

## Notes

[Any additional notes]