# Patch Boundary

## Allowed Files

以下文件可以修改：

- [file path 1]
- [file path 2]
- [file path 3]

### Allowed New Files (Optional)

以下新文件允许创建（需提前列入）：

- [new file path 1] (仅限测试文件)
- [new file path 2] (仅限测试文件)

**注意**：如需新增文件，必须在 Analyze 阶段提前规划并列入此处。

---

## Forbidden Files

以下文件禁止修改：

- [file path 1]
- [file path 2]
- [file path 3]

---

## Forbidden Behaviors

以下行为禁止执行：

- ❌ 重构代码结构
- ❌ 顺手优化
- ❌ 架构改进
- ❌ 改动无关模块
- ❌ 添加新业务功能
- ❌ 修改代码风格
- ❌ 重命名变量/函数
- ❌ 删除现有文件

### New Files Rule (v1.1)

- ❌ **默认禁止新增业务文件**
- ✅ 新增测试文件必须提前列入 Allowed New Files
- ❌ 未列入 Allowed 的新增文件 → VERIFY 时一律 FAIL

---

## Stop Conditions

遇到以下情况必须立即停止并报告：

- 🛑 发现需要修改 Forbidden Files
- 🛑 发现问题无法在 Allowed Files 内解决
- 🛑 发现新 regression 风险
- 🛑 发现需求理解偏差
- 🛑 发现需要扩大修改范围
- 🛑 发现需要新增业务文件（未提前列入 Allowed）
- 🛑 发现需要修改测试文件外的其他文件

---

## Notes

[Any special notes for this patch]

---

## Generated From

- ANALYZE_REPORT.md: [引用版本]
- Generated: [timestamp]