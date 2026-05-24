# Patch Boundary

## Allowed Files

以下文件可以修改：

- `src/components/ProductEditor.vue`
- `src/utils/ext_save_data.js`

---

## Forbidden Files

以下文件禁止修改：

- `src/store/variants.js` - 核心状态管理，风险高
- `workflow/` - 工作流模块
- `warehouse/` - 仓库模块
- `products/main_table.sql` - 产品主表
- `src/store/index.js` - 状态管理入口

---

## Forbidden Behaviors

以下行为禁止执行：

- ❌ 重构代码结构
- ❌ 顺手优化其他组件
- ❌ 架构改进
- ❌ 改动无关模块
- ❌ 添加新功能
- ❌ 修改代码风格
- ❌ 重命名变量/函数
- ❌ 添加新文件
- ❌ 删除现有文件
- ❌ 修改 variants 核心状态结构

---

## Stop Conditions

遇到以下情况必须立即停止并报告：

- 🛑 发现需要修改 `src/store/variants.js`
- 🛑 发现需要修改 workflow/warehouse 模块
- 🛑 发现问题无法在 Allowed Files 内解决
- 🛑 发现新 regression 风险
- 🛑 发现需求理解偏差
- 🛑 发现需要扩大修改范围
- 🛑 发现需要新增文件

---

## Notes

根据 RR-001，修改时必须同步检查：
- update_ary 函数调用是否正常
- insert_ary 函数调用是否正常
- ext_save_data 数据处理是否完整

---

## Generated From

- ANALYZE_REPORT.md: v3 (LOCKED)
- Generated: 2024-05-24 10:50