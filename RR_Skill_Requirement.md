# AI Patch Governance Skill（RR Skill）需求文档

---

## Historical Document Notice

**本文件是历史需求文档，不是当前执行规则。**

当前执行规则以以下文件为准：
- `RR_SKILL.md` - 核心 Skill Prompt 规范
- `templates/` - 模板文件
- `docs/WORKFLOW.md` - 详细工作流说明
- `VERSION.md` - 版本记录

本文件仅作为历史参考，记录了最初的设计思路和需求来源。

---

## 一、项目目标

构建一个用于 AI Coding 的「Patch Governance Skill（补丁治理 Skill）」。

目标不是：

- 让 AI 更会写代码

而是：

- 让 AI 在增量修改（Patch）阶段：
  - 不乱改
  - 不扩大问题范围
  - 不破坏已有系统
  - 不发生上下文漂移
  - 不发生架构侵蚀

核心思想：

AI 在真正修改代码前，必须先：

1. 分析问题
2. 约束修改范围
3. 识别 regression
4. 生成 patch rules
5. 生成 task packet
6. 经人类确认后才能开发

该 Skill 的本质是：

AI Patch Governor

而不是普通 Prompt Generator。

---

# 二、核心设计理念

当前 AI Coding 的核心问题：

AI 在 patch 阶段：

- 会疯狂涂改
- 会扩大修改半径
- 会局部修复破坏全局
- 会进行无关重构
- 会脑补 architecture
- 会发生 context drift

因此：

真正需要治理的是：

AI 的“思考过程”

而不是仅治理：

AI 的“代码输出”

所以：

RR Skill 必须从“问题分析阶段”就介入。

---

# 三、系统定位

该系统属于：

AI Coding Middleware

位于：

Human
↕
RR Skill
↕
LLM / Codex / Claude / Cursor Agent
↕
Codebase

之间。

---

# 四、核心工作流

## Workflow

发现问题
↓
触发 RR Analyze
↓
Skill 接管分析过程
↓
输出：
- 问题归类
- 修改边界
- 风险点
- 是否属于 regression
- 是否需要新增 RR
↓
人类确认
↓
触发 RR Commit
↓
生成：
- PATCH_RULES.md
- AI_TASK_PACKET.md
- PATCH_PLAN.md
↓
触发 RR Implement
↓
AI 严格依据 Patch Packet 开发
↓
输出：
- 修改文件
- 风险
- AC 覆盖
- 未验证项
↓
人类 Review

---

# 五、核心阶段设计

# Phase 1：RR Analyze（最关键）

## 目标

禁止 AI 直接开始改代码。

必须先：

分析问题

---

## 输入

用户自然语言：

- “这里回显丢了”
- “这里 warehouse 不同步”
- “这里 save pipeline 有问题”

---

## 输出结构

### 1. Problem Classification

问题分类：

- UI Bug
- State Bug
- Save Pipeline Bug
- Render Bug
- Workflow Bug
- Architecture Bug
- Regression

---

### 2. Impact Radius

分析：

可能影响哪些链路

例如：

涉及：
- tmp_1
- ext_save_data
- variants

不涉及：
- workflow
- products 主表
- warehouse architecture

---

### 3. Patch Boundary

生成：

允许修改：
禁止修改：

例如：

Allowed:
- tmp_1
- ext_save_data
- update_ary

Forbidden:
- workflow
- warehouse structure
- products main table
- architecture refactor

---

### 4. Regression Detection

分析：

这是：
- 新问题
- 已知 regression
- 需求变更
- 架构缺陷

---

### 5. Patch Strategy

生成：

最小修改方案

禁止：

大范围重构

---

### 6. Risk Analysis

分析：

可能误伤：
- insert_ary
- copy mode
- new mode
- save pipeline

---

## Phase 1 重要规则

### Rule 1

Analyze 阶段：

禁止修改代码

### Rule 2

Analyze 阶段：

禁止生成 patch

### Rule 3

Analyze 阶段：

禁止 architecture 优化

### Rule 4

如果问题无法局部修复：

必须输出：

当前问题无法在限定边界内解决，需要升级为 architecture issue。

而不是自动扩大修改范围。

---

# 六、Phase 2：RR Commit

## 触发条件

用户确认：

“确认需求，触发 RR”

---

## 目标

把 Analyze 结果结构化沉淀为长期规则。

---

## 生成文件

### 1. PATCH_RULES.md

目标：

沉淀以后 AI 不允许再犯的错误。

示例结构：

```md
# PATCH RULES

## RR-001

修改 variants 时：

必须同步检查：
- update_ary
- insert_ary
- ext_save_data

---

## RR-002

修改 tmp_1 时：

必须检查：
- JS replace
- input name
- default object

---

## RR-003

涉及 warehouse 时：

禁止只修改基础区。

必须同步检查：
- ProInfo
- warehouseSet
```

---

### 2. PATCH_PLAN.md

目标：

生成本次 patch 的正式修改计划。

内容：

- 问题
- 影响范围
- 允许修改文件
- 禁止修改文件
- 风险
- 最小 patch 策略

---

### 3. AI_TASK_PACKET.md

目标：

生成真正给 AI 的开发包。

内容：

- 本次目标
- 允许修改范围
- 禁止修改范围
- 必须遵守的 RR
- 风险项
- 输出要求

---

# 七、Phase 3：RR Implement

## 目标

真正开始 patch。

---

## 输入

只能来自：

AI_TASK_PACKET.md

禁止：

自由发挥

---

## Patch 规则

### Rule 1

AI 只能修改：

Allowed Files

### Rule 2

AI 禁止：

主动重构

### Rule 3

AI 禁止：

扩大 patch 半径

### Rule 4

如果 patch 失败：

禁止继续 patch。

必须：

回到 RR Analyze

重新分析。

---

# 八、Patch Discipline（核心）

系统核心理念：

限制 AI 修改冲动

而不是：

提高 AI 自由度

---

# 九、长期记忆策略

禁止：

无限聊天上下文

---

## 长期层

### PATCH_RULES.md

长期存在。

用于：

Regression Prevention

---

## 中期层

### PATCH_PLAN.md

当前 patch 生命周期有效。

---

## 短期层

### AI_TASK_PACKET.md

当前开发任务有效。

开发结束可归档。

---

# 十、系统触发机制

## Trigger 1

“确认需求，触发 RR”

进入：

RR Analyze

---

## Trigger 2

“确认 patch plan”

进入：

RR Commit

---

## Trigger 3

“开始开发”

进入：

RR Implement

---

# 十一、需求确认机制（重要）

RR Analyze 必须支持：

- 多轮校对
- 多轮纠正
- 多轮边界缩小
- 多轮风险确认

流程：

问题提出
↓
RR Analyze Draft v1
↓
用户纠正
↓
RR Analyze Draft v2
↓
用户补充
↓
RR Analyze Draft v3
↓
用户确认
↓
LOCK Analyze Result
↓
进入 RR Commit

---

## Requirement Freeze（需求冻结）

Analyze Result 一旦 Lock：

开发阶段禁止：

- 重新解释需求
- 扩大修改范围
- 增加新 architecture
- 自由重构

如需新增修改：

必须重新进入：

RR Analyze

---

# 十二、Regression Rule（RR）设计

RR 的目标不是：

定义功能

而是：

防止 AI 再次犯错。

RR 属于：

Regression Prevention Layer

---

## RR 示例

```md
RR-014

修改 tmp_1 时：

必须同步检查：
- replace chain
- ext_save_data
- variants name structure
```

---

# 十三、Patch Boundary（核心）

系统核心：

限制 AI 的 patch 半径。

例如：

Allowed:
- tmp_1
- ext_save_data
- update_ary

Forbidden:
- workflow
- warehouse architecture
- products 主表
- architecture refactor

---

# 十四、Patch Failure 策略

如果 patch 后：

- bug 扩大
- 修改半径失控
- 新增 regression
- architecture drift

则：

禁止继续 patch。

必须：

1. 回滚当前 patch
2. 重新进入 RR Analyze
3. 重新生成 Patch Boundary

禁止：

无限 patch loop

---

# 十五、系统目标

最终实现：

AI 不再：

- 疯狂涂改
- 无限 patch
- 顺手优化
- architecture drift
- patch explosion

而是：

像成熟工程师一样：

- 小范围修改
- 风险可控
- patch 可追踪
- regression 可治理

---

# 十六、核心原则

## Principle 1

Analyze First
Patch Later

---

## Principle 2

Patch Boundary > Prompt Engineering

---

## Principle 3

Regression Rules > Long Context

---

## Principle 4

限制 AI 的思考范围，比增强 AI 更重要。

---

# 十七、最终定位

该 Skill 不属于：

Code Generator

而属于：

AI Software Governance System
