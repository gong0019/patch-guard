# Patch Plan

## Problem

编辑产品页面时，variants 数据的回显丢失，保存后数据不完整。

分类: State Bug

---

## Scope

### Allowed Files

- `src/components/ProductEditor.vue`
- `src/utils/ext_save_data.js`

### Forbidden Files

- `src/store/variants.js`
- `workflow/`
- `warehouse/`
- `products/main_table.sql`

---

## Minimal Changes

### `src/components/ProductEditor.vue`

修改内容:
- 检查 variants 回显逻辑，确保数据正确绑定
- 修复数据初始化时机问题
- 不修改组件整体结构

### `src/utils/ext_save_data.js`

修改内容:
- 确保 variants 数据在保存时正确处理
- 检查数据序列化逻辑
- 不修改核心数据处理流程

---

## Pre-Check

修改前需要先检查：

- [ ] 确认 variants 数据在编辑器中的绑定方式
- [ ] 确认 ext_save_data.js 的数据处理流程
- [ ] 确认 update_ary 和 insert_ary 函数调用位置

---

## Post-Check

修改后需要验证：

- [ ] 编辑页面 variants 回显正常
- [ ] 保存后 variants 数据完整
- [ ] update_ary 调用正常
- [ ] insert_ary 调用正常
- [ ] 其他使用 variants 的页面不受影响

---

## Risks

- update_ary 调用失效: 可能导致数据更新失败
- insert_ary 调用失效: 可能导致新增数据失败
- 其他页面影响: 需检查是否有其他组件依赖相同逻辑

---

## Related RR Rules

需要遵守的已有规则：

- **RR-001**: 修改 variants 时必须检查 update_ary、insert_ary、ext_save_data

---

## Generated From

- ANALYZE_REPORT.md: v3 (LOCKED)
- BOUNDARY.md: v1
- Generated: 2024-05-24 10:55