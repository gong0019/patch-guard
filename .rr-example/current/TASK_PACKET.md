# AI Task Packet

## Objective

修复 ProductEditor.vue 中 variants 数据回显丢失问题，确保 ext_save_data.js 正确处理数据。

---

## Allowed Scope

### Files

以下文件可以修改：

- `src/components/ProductEditor.vue`
- `src/utils/ext_save_data.js`

### Behaviors

仅允许：
- 最小化修改回显逻辑
- 修复数据处理问题
- 保持现有代码结构

---

## Forbidden Scope

### Files

以下文件禁止修改：

- `src/store/variants.js`
- `workflow/`
- `warehouse/`
- `products/main_table.sql`
- `src/store/index.js`

### Behaviors

禁止执行：
- ❌ 重构
- ❌ 优化
- ❌ 架构改动
- ❌ 改动无关模块
- ❌ 添加新功能
- ❌ 修改代码风格
- ❌ 重命名

---

## Must Follow

### Related RR Rules

**RR-001**: 修改 variants 时必须检查 update_ary、insert_ary、ext_save_data

执行前确认：
- update_ary 函数调用位置
- insert_ary 函数调用位置
- ext_save_data 数据处理流程

### Pre-Check Items

- 确认 variants 数据在编辑器中的绑定方式
- 确认 ext_save_data.js 的数据处理流程

---

## Risks to Watch

- ⚠️ update_ary 调用失效
- ⚠️ insert_ary 调用失效
- ⚠️ 其他页面受影响

---

## Stop Conditions

遇到以下情况必须立即停止：

- 🛑 发现需要修改 `src/store/variants.js`
- 🛑 发现需要修改 workflow/warehouse 模块
- 🛑 发现问题无法在 Allowed Files 内解决
- 🛑 发现新 regression 风险
- 🛑 发现需求理解偏差

---

## Output Requirements

修改完成后必须输出：

1. ✅ 修改文件列表
2. ✅ 每个文件的修改摘要
3. ✅ 未验证项列表
4. ✅ 风险项列表
5. ✅ 是否触达 Stop Conditions

---

## Generated From

- ANALYZE_REPORT.md: v3 (LOCKED)
- BOUNDARY.md: v1
- PATCH_PLAN.md: v1
- Generated: 2024-05-24 11:00