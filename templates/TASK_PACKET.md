# TASK_PACKET.md

## Generation Rules (v3.0)

**TASK_PACKET 只有在 LOCKED_PLAN 后才能成为可执行工单。**

如果还在 Analyze Draft 阶段：
- ❌ 不得生成 TASK_PACKET.md（可执行）
- ✅ 只能生成 TASK_PACKET_DRAFT.md（草案）
- ✅ 或不生成

**禁止在未确认状态下生成可执行 TASK_PACKET。**

---

## P0/P1/P2 Separation (v3.0)

### Priority Definition

| Priority | Definition | Allowed Location |
|----------|------------|------------------|
| P0 | 本轮必须实现的核心功能 | TASK_PACKET.md |
| P1 | 后续可实现的功能 | Future Work（不纳入 TASK_PACKET） |
| P2 | 明确排除的功能 | Out of Scope（不纳入文档） |

### Hard Rule

**TASK_PACKET 只能包含 P0。**

禁止：
- ❌ 把 P1 混入 TASK_PACKET
- ❌ 把 P2 混入 TASK_PACKET
- ❌ 在 TASK_PACKET 中标注 P1/P2

如果检测到 P1/P2 在 TASK_PACKET：
- 返回 Analyze
- 移除 P1/P2
- 用户确认后才能 Lock

---

## Objective

[本次修改的具体目标，一句话描述 - 只包含 P0]

---

## Allowed Scope

### Files

以下文件可以修改：

- [file 1]
- [file 2]

### Allowed New Files

以下新文件允许创建（需提前列入）：

- [new file path 1] (仅限测试文件)
- [new file path 2] (仅限测试文件)

**注意**：如需新增文件，必须在 Analyze 阶段提前规划并列入此处。

**New Files Rule (v1.1)**：
- ❌ **默认禁止新增业务文件**
- ✅ 新增测试文件必须提前列入 Allowed New Files
- ❌ 未列入 Allowed 的新增文件 → VERIFY 时一律 FAIL

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
- ❌ 添加新业务功能
- ❌ 修改代码风格
- ❌ 重命名
- ❌ 新增未列入 Allowed 的文件

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

## Stop Conditions (v1.1)

遇到以下情况必须立即停止：

- 🛑 发现需要修改 Forbidden Files
- 🛑 发现问题无法在 Allowed Files 内解决
- 🛑 发现新 regression 风险
- 🛑 发现需求理解偏差
- 🛑 发现需要扩大修改范围
- 🛑 **发现需要新增文件但未在 Allowed New Files 中声明**
- 🛑 **发现需要新增业务文件（未提前列入 Allowed）**

**硬规则**：如果实现需要新增文件但未在 Boundary / Task Packet 中声明，必须 STOP 并返回 rr analyze。

---

## Output Requirements

修改完成后必须输出：

1. ✅ 修改文件列表
2. ✅ 每个文件的修改摘要
3. ✅ 新增文件列表（如有）
4. ✅ 未验证项列表
5. ✅ 风险项列表
6. ✅ 是否触达 Stop Conditions

---

## Generated From

- ANALYZE_REPORT.md: [引用版本]
- BOUNDARY.md: [引用版本]
- PATCH_PLAN.md: [引用版本]
- Generated: [timestamp]