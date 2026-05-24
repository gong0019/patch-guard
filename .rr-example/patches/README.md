# Patch Session Example

本目录是一个完整的 patch session 示例，展示 PatchGuard v3.0 的文件结构。

---

## Patch ID

`sample-comment-replies`

---

## Files

| File | Purpose |
|------|---------|
| ANALYZE_REPORT.md | 分析报告，包含 Problem Understanding + Patch ID |
| BOUNDARY.md | Allowed/Forbidden Files/Behaviors/Stop Conditions |
| PATCH_PLAN.md | 修改计划 |
| TASK_PACKET.md | AI 执行任务包 |
| VERIFY_REPORT.md | 验证报告 |
| CURRENT_STATE.md | 当前阶段状态 |
| LAST_LOCKED_PLAN.md | 锁定的分析结果 |

---

## Key Points (v3.0)

1. **每个 patch 独立目录**：`.rr/patches/<patch-id>/`
2. **不再使用 .rr/current/ 和 .rr/state/**
3. **CURRENT_STATE.md 和 LAST_LOCKED_PLAN.md 属于 patch 目录**
4. **Patch ID 必须用户确认**

---

## Directory Structure

```
.rr-example/
├── patches/
│   └── sample-comment-replies/   # patch session 示例
│       ├── ANALYZE_REPORT.md
│       ├── BOUNDARY.md
│       ├── CURRENT_STATE.md
│       ├── LAST_LOCKED_PLAN.md
│       ├── PATCH_PLAN.md
│       ├── TASK_PACKET.md
│       └── VERIFY_REPORT.md
│
├── rules/
│   ├── PATCH_RULES.md            # 全局长期规则
│   └── CORE_MODULES.md           # 核心模块列表
│
└── archive/
    └── README.md                  # 归档规范
```