# AI Task Packet

## Objective

[本次修改的具体目标，一句话描述]

---

## Allowed Scope

### Files

以下文件可以修改：

- [file 1]
- [file 2]

### Behaviors

仅允许：
- 最小化修改
- 针对性修复
- 保持现有代码结构

---

## Forbidden Scope

### Files

以下文件禁止修改：

- [file 1]
- [file 2]

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

[引用相关 RR Rules]

- RR-XXX: [规则内容摘要]
- 或: 无特殊规则

### Pre-Check Items

[从 PATCH_PLAN.md 引用 Pre-Check]

---

## Risks to Watch

- ⚠️ [风险 1]
- ⚠️ [风险 2]

---

## Stop Conditions

遇到以下情况必须立即停止：

- 🛑 发现需要修改 Forbidden Files
- 🛑 发现问题无法在 Allowed Files 内解决
- 🛑 发现新 regression 风险
- 🛑 发现需求理解偏差
- 🛑 发现需要扩大修改范围

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

- ANALYZE_REPORT.md: [引用版本]
- BOUNDARY.md: [引用版本]
- PATCH_PLAN.md: [引用版本]
- Generated: [timestamp]