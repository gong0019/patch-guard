# RR Analyze Report

## Status
LOCKED

---

## Problem

用户报告：编辑产品页面时，variants 数据的回显丢失，保存后数据不完整。

---

## Problem Classification

选择一项：

- [ ] UI Bug
- [x] State Bug
- [ ] Logic Bug
- [ ] Save Pipeline Bug
- [ ] Render Bug
- [ ] Workflow Bug
- [ ] Architecture Bug
- [ ] Regression

分类结果: **State Bug** - variants 数据状态在编辑流程中丢失

---

## Impact Radius

### 涉及

- `src/components/ProductEditor.vue` - 编辑器组件
- `src/store/variants.js` - variants 状态管理
- `src/utils/ext_save_data.js` - 保存数据处理

### 不涉及

- `workflow/` - 工作流模块
- `warehouse/` - 仓库模块
- `products/main_table.sql` - 产品主表结构

---

## Regression Detection

选择一项：

- [x] New Issue - 新发现的问题
- [ ] Known Regression - 已知的回归问题
- [ ] Requirement Change - 需求变更导致
- [ ] Architecture Defect - 架构缺陷

检测结果: **New Issue** - 最近一次修改 variants 显示逻辑后出现

---

## Risk Analysis

### 可能误伤

- `update_ary` 函数 - 调用 variants 数据更新
- `insert_ary` 函数 - 新增 variants 数据
- 其他使用 variants 的页面组件

### 关联已有 RR

- **RR-001**: 修改 variants 时必须检查 update_ary、insert_ary、ext_save_data
- RR-002: tmp_1 相关检查（本次不涉及）

---

## Minimal Fix Strategy

**方案**：
1. 在 ProductEditor.vue 中检查 variants 数据回显逻辑
2. 确保 ext_save_data.js 正确处理 variants 数据
3. 不修改 variants.js 的核心状态结构

**禁止**:
- 大范围重构 variants 状态管理
- 顺手优化其他组件
- 架构改动

**允许**:
- 最小化修改回显逻辑
- 保持现有结构

---

## Version History

| Version | Time | Change |
|---------|------|--------|
| v1 | 2024-05-24 10:00 | initial analysis: identified as State Bug |
| v2 | 2024-05-24 10:15 | user correction: 添加 ext_save_data.js 到涉及列表 |
| v3 | 2024-05-24 10:30 | user补充: 确认不涉及 workflow 和 warehouse |
| LOCKED | 2024-05-24 10:45 | 用户确认锁定 |

---

## Notes

此问题需要遵守 RR-001 规则，修改时同步检查 update_ary 和 insert_ary。