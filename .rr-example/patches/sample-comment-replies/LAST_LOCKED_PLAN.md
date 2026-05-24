# Last Locked Plan

## Lock Info

| Field | Value |
|-------|-------|
| locked_at | 2024-05-24 10:45 |
| locked_by | user confirmation |
| analyze_version | v3 |

## Original Problem

用户报告：编辑产品页面时，variants 数据的回显丢失，保存后数据不完整。

## Locked Problem Classification

State Bug - variants 数据状态在编辑流程中丢失

## Locked Impact Radius

涉及:
- `src/components/ProductEditor.vue`
- `src/store/variants.js`
- `src/utils/ext_save_data.js`

不涉及:
- `workflow/`
- `warehouse/`
- `products/main_table.sql`

## Locked Minimal Fix Strategy

修复 ProductEditor.vue 中 variants 数据回显绑定逻辑，确保 ext_save_data.js 正确处理数据。禁止重构 variants 状态管理。

## Locked Allowed Files

- `src/components/ProductEditor.vue`
- `src/utils/ext_save_data.js`

## Locked Forbidden Files

- `src/store/variants.js`
- `workflow/`
- `warehouse/`
- `products/main_table.sql`
- `src/store/index.js`

## Locked Stop Conditions

- 发现需要修改 Forbidden Files
- 发现问题无法在 Allowed Files 内解决
- 发现新 regression 风险
- 发现需要扩大修改范围

## Change History

| Version | Change | Reason |
|---------|--------|--------|
| v1 | initial | 初次分析 |
| v2 | 增加 ext_save_data.js | 用户补充 |
| v3 | 确认不涉及 workflow/warehouse | 用户确认 |

## Notes

此锁定计划在 Implement 阶段禁止重新解释。如有偏差必须 STOP 并返回 rr analyze。