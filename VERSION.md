# RR Skill Version Log

## v1.1.1 (2024-05-24)

### Release Summary

V1.1 一致性和可读性修正，不增加新功能。

**修正内容**：
- Markdown 物理换行确认正常（已验证标题、列表独立行）
- TASK_PACKET.md 同步新增文件规则（Allowed New Files、STOP 条件）
- WORKFLOW Summary 状态语义修正（PASS/WARNING/FAIL 正确表达）
- VERSION 历史规则歧义说明（v1.0 行为以最新版本为准）

---

### Fixes

#### 1. Markdown 物理换行确认

已验证所有文件格式正确：
- 标题独立一行
- 列表项独立一行
- 表格正常 Markdown 格式
- 代码块保留换行

无单行 Markdown 文件。

#### 2. TASK_PACKET.md 新增文件规则

增加内容：
- **Allowed New Files** 部分
- **New Files Rule (v1.1)** 说明
- **Stop Conditions (v1.1)** 增加新增文件条件
- **硬规则**：实现需要新增文件但未声明 → STOP 并返回 rr analyze
- Output Requirements 增加"新增文件列表"

#### 3. WORKFLOW Summary 状态语义修正

v1.1.0: `Verify | 检查合规，PASS 才提交`

v1.1.1: `Verify | PASS 可进入人工审查；WARNING 必须人工验证；FAIL 必须回滚`

消除过度简化表达，明确三种状态的真实含义。

#### 4. VERSION 历史规则歧义说明

在 v1.0.0 部分前增加：

```
## Historical Records Notice

以下为历史版本记录。当前行为以 v1.1.0/v1.1.1 Status Definition / Hard Rules 为准。
```

---

### File Changes

| File | Change |
|------|--------|
| templates/TASK_PACKET.md | 增加 Allowed New Files、New Files Rule、Stop Conditions v1.1 |
| docs/WORKFLOW.md | Summary 表格状态语义修正 |
| .rr-example/current/TASK_PACKET.md | 同步 Allowed New Files 规则 |
| VERSION.md | 增加历史规则歧义说明、v1.1.1 记录 |

---

## v1.1.0 (2024-05-24)

### Release Summary

V1 协议修正版本，修复不一致和可读性问题，不增加新功能。

**修正内容**：
- Markdown 格式确认正常（已验证多行格式）
- Verify 状态定义修复（PASS/WARNING/FAIL 条件调整）
- 示例状态修复（Unverified Items 存在时改为 WARNING）
- Boundary 新增文件规则修复（默认禁止，需提前列入）
- Unverified Items 不得 PASS 硬规则
- 文档说明 V1 是 Prompt Protocol，不是自动验证工具

---

### Fixes

#### 1. Verify 状态定义修复

| Status | Condition (v1.0) | Condition (v1.1) |
|--------|------------------|------------------|
| PASS | 全部 Allowed，无 Forbidden | 无越界、无 Forbidden、**无未验证关键项** |
| WARNING | 全部 Allowed，有未验证项 | 无越界、无 Forbidden，**但存在未验证项** |
| FAIL | 超出边界或触碰 Forbidden | 触碰 Forbidden、超出 Allowed、违反 Locked Plan、或新增未 Allowed 文件 |

**硬规则**：如果存在 Unverified Items，状态不能是 PASS，只能是 WARNING 或 FAIL。

#### 2. VERIFY_REPORT.md 模板强化

新增字段：
- **Evidence** - 修改证据记录（Diff Source、Modified Lines、Review Method）
- **Manual Verification Required** - 需人工验证的项目表
- **New Files Created** - 新增文件及其 Allowed 状态
- **Final Recommendation** - 最终提交建议

#### 3. BOUNDARY.md 新增文件规则

v1.0: ❌ 添加新文件（绝对禁止）

v1.1:
- ❌ **默认禁止新增业务文件**
- ✅ 新增测试文件必须提前列入 Allowed New Files
- ❌ 未列入 Allowed 的新增文件 → VERIFY 时一律 FAIL

新增 `Allowed New Files (Optional)` 部分。

#### 4. V1 Limitations 说明

在 RR_SKILL.md、QUICK_START.md、WORKFLOW.md 中明确说明：
- **V1 是 Prompt Protocol，不是自动验证工具**
- **rr verify 仍然需要人工审查**
- **PASS 不代表业务完全正确**，只代表边界检查通过
- **WARNING 必须人工验证后才能提交**

#### 5. 示例文件修复

`.rr-example/current/VERIFY_REPORT.md`：
- 状态从 PASS 改为 WARNING（存在 Unverified Items）
- 增加 Evidence 和 Manual Verification Required 字段

---

### File Changes

| File | Change |
|------|--------|
| RR_SKILL.md | Phase 4 状态定义修复、BOUNDARY.md Format 更新、Hard Rules 增加 |
| templates/VERIFY_REPORT.md | 增加 Evidence、Manual Verification Required、Final Recommendation |
| templates/BOUNDARY.md | 增加 Allowed New Files 部分、New Files Rule |
| .rr-example/current/VERIFY_REPORT.md | 状态改为 WARNING、增加新字段 |
| .rr-example/current/BOUNDARY.md | 增加 Allowed New Files 部分 |
| docs/QUICK_START.md | 增加 V1 Limitations、Status Definition v1.1 |
| docs/WORKFLOW.md | 增加 V1 Limitations、Phase 4 Hard Rules、更新 Best Practices |
| VERSION.md | 增加 v1.1.0 记录 |

---

### Status Definition (v1.1)

| Status | Condition | Action |
|--------|-----------|--------|
| PASS | 无越界、无 Forbidden、**无未验证关键项** | ✅ 可提交（仍需人工审查） |
| WARNING | 无越界、无 Forbidden，**但存在未验证项** | ⚠️ **必须人工验证后才能提交** |
| FAIL | 触碰 Forbidden、超出 Allowed、违反 Locked Plan、或新增未 Allowed 文件 | ❌ 必须回滚，返回 Phase 1 |

---

### Hard Rules (v1.1)

1. 如果存在 Unverified Items，状态不能是 PASS，只能是 WARNING 或 FAIL
2. 未列入 Allowed 的新增文件一律 FAIL
3. PASS 不代表业务完全正确，只代表边界检查通过
4. WARNING 必须人工验证后才能提交

---

## Historical Records Notice

**以下为历史版本记录。当前行为以 v1.1.0/v1.1.1 Status Definition / Hard Rules 为准。**

v1.0.0 的状态定义和规则已被 v1.1.0 修正，请参考最新版本。

---

## v1.0.0 (2024-05-24)

### Release Summary

首个正式版本，聚焦核心流程约束和模板文件系统。

**设计决策**：
- 形态：通用 Prompt 规范（Markdown），任何 AI 工具都能加载执行
- AI 接口：无内置 AI，输出模板文件供用户在 Cursor/Claude/Copilot 中使用
- 规则存储：项目级 `.rr/rules/` 和 `.rr/current/` 分离
- 辅助工具：v2 规划，不进入 v1

---

### Features

#### 1. 四阶段工作流

| Phase | Trigger | Output | Core Rule |
|-------|---------|--------|-----------|
| Analyze | `RR 分析` | ANALYZE_REPORT.md | 禁止修改代码 |
| Commit | `确认计划` | BOUNDARY.md, PATCH_PLAN.md, TASK_PACKET.md | 禁止重新解释需求 |
| Implement | `开始执行` | 修改文件 | 只修改 Allowed Files |
| Verify | `RR 验证` | VERIFY_REPORT.md | 检查边界合规 |

#### 2. LOCKED_PLAN 机制

- Analyze 阶段支持多轮校对（v1 → v2 → v3 → LOCKED）
- 锁定后禁止：
  - 重新解释需求
  - 扩大 Allowed Files
  - 减少 Forbidden Files
  - 增加新 architecture
  - 自由重构
- 如需变更：必须输入 `停止，重新分析` 返回 Phase 1

#### 3. BOUNDARY.md 边界约束

包含四个核心部分：
- **Allowed Files** - 允许修改的文件列表
- **Forbidden Files** - 禁止修改的文件列表
- **Forbidden Behaviors** - 禁止的行为列表（重构、优化、架构改动等）
- **Stop Conditions** - 必须立即停止的条件列表

#### 4. 目录分离

| Directory | Lifecycle | Purpose |
|-----------|-----------|---------|
| `.rr/rules/` | 永久 | 长期 Regression Rules，跨 patch 复用 |
| `.rr/current/` | 当前 patch | 当前 patch 生命周期文件，结束后可归档删除 |

**写入长期规则条件**：
- 重大 regression，未来可能复现
- 典型错误模式，其他 AI 可能重犯
- 只有用户明确要求时才写入

#### 5. RR Verify 验证阶段

输出 VERIFY_REPORT.md 包含：
- 实际修改文件列表
- Allowed Files 检查结果
- Forbidden Files 检查结果
- Forbidden Behaviors 检查结果
- 未验证项列表
- 风险项列表
- 状态判定（PASS/WARNING/FAIL）

**状态定义**：
| Status | Condition | Action |
|--------|-----------|--------|
| PASS | 全部 Allowed，无 Forbidden，风险可控 | ✅ 可提交 |
| WARNING | 全部 Allowed，但有未验证项 | ⚠️ 需补充验证后提交 |
| FAIL | 有超出 Allowed 或触碰 Forbidden | ❌ 必须回滚，返回 Phase 1 |

#### 6. Emergency Stop

触发词：`停止，重新分析`

动作：
1. 立即停止当前操作
2. 回滚未提交的修改（如需要）
3. 返回 Phase 1
4. 重新生成 ANALYZE_REPORT.md (DRAFT)

---

### File Structure

```
PatchGuard/
├── RR_SKILL.md                    # 核心 Skill Prompt 规范
│                                  # - Role Definition
│                                  # - Trigger Keywords
│                                  # - Four Phase Details
│                                  # - LOCKED_PLAN Rules
│                                  # - Directory Convention
│                                  # - Core Principles
│
├── templates/                     # 模板文件（用户复制到 .rr/current/）
│   ├── ANALYZE_REPORT.md          # 分析报告模板
│   │                              # - Status (DRAFT/LOCKED)
│   │                              # - Problem Classification
│   │                              # - Impact Radius
│   │                              # - Regression Detection
│   │                              # - Risk Analysis
│   │                              # - Minimal Fix Strategy
│   │                              # - Version History
│   │
│   ├── BOUNDARY.md                # 修改边界模板
│   │                              # - Allowed Files
│   │                              # - Forbidden Files
│   │                              # - Forbidden Behaviors (8项)
│   │                              # - Stop Conditions (7项)
│   │
│   ├── PATCH_PLAN.md              # Patch 计划模板
│   │                              # - Problem (引用)
│   │                              # - Scope (引用)
│   │                              # - Minimal Changes
│   │                              # - Pre-Check / Post-Check
│   │                              # - Risks
│   │                              # - Related RR Rules
│   │
│   ├── TASK_PACKET.md             # AI 任务包模板
│   │                              # - Objective
│   │                              # - Allowed/Forbidden Scope
│   │                              # - Must Follow (RR Rules)
│   │                              # - Risks to Watch
│   │                              # - Stop Conditions
│   │                              # - Output Requirements
│   │
│   └── VERIFY_REPORT.md           # 验证报告模板
│                                  # - Status (PASS/WARNING/FAIL)
│                                  # - Modified Files
│                                  # - Boundary Check (三部分)
│                                  # - Unverified Items
│                                  # - Risks
│                                  # - Recommendation
│
├── .rr-example/                   # 完整工作流示例
│   ├── rules/
│   │   └── PATCH_RULES.md         # 长期规则示例 (RR-001~RR-005)
│   │                              # - variants 修改规则
│   │                              # - tmp_1 修改规则
│   │                              # - warehouse 修改规则
│   │                              # - save pipeline 规则
│   │                              # - workflow 规则
│   │
│   └── current/                   # 当前 patch 示例（全部填充）
│       ├── ANALYZE_REPORT.md      # 示例：variants 回显丢失问题
│       ├── BOUNDARY.md            # 示例：ProductEditor.vue + ext_save_data.js
│       ├── PATCH_PLAN.md          # 示例：最小修改方案
│       ├── TASK_PACKET.md         # 示例：具体任务约束
│       └── VERIFY_REPORT.md       # 示例：PASS 状态验证结果
│
├── docs/
│   ├── QUICK_START.md             # 快速上手指南
│   │                              # - Setup (3步)
│   │                              # - Basic Usage (6步)
│   │                              # - Trigger Keywords Reference
│   │                              # - Directory Structure
│   │                              # - Best Practices
│   │
│   └── WORKFLOW.md                # 详细工作流说明
│                                  # - 四阶段详细说明
│                                  # - 每阶段 Rules 表格
│                                  # - Stop Conditions 详细
│                                  # - Directory Lifecycle
│                                  # - Best Practices
│                                  # - Common Mistakes
│
├── README.md                      # 项目说明
│                                  # - What is RR Skill
│                                  # - Quick Start
│                                  # - Structure
│                                  # - Four Phases
│                                  # - Usage in Different AI Tools
│
├── VERSION.md                     # 本文件 - 版本管理
│
└── RR_Skill_Requirement.md        # 需求文档（原始输入）
```

---

### Trigger Keywords

| Trigger | Phase | Action |
|---------|-------|--------|
| `RR 分析` / `rr analyze` | Phase 1 | 分析问题，输出 ANALYZE_REPORT.md |
| `确认计划` / `rr commit` | Phase 2 | 锁定计划，生成边界/任务文件 |
| `开始执行` / `rr implement` | Phase 3 | 按 TASK_PACKET.md 约束修改代码 |
| `RR 验证` / `rr verify` | Phase 4 | 检查边界合规性，输出 VERIFY_REPORT.md |
| `停止，重新分析` | Any | 立即停止，返回 Phase 1 |

---

### Problem Classification Types

| Type | Description |
|------|-------------|
| UI Bug | 用户界面显示问题 |
| State Bug | 数据状态管理问题 |
| Logic Bug | 业务逻辑错误 |
| Save Pipeline Bug | 数据保存流程问题 |
| Render Bug | 渲染/绘制问题 |
| Workflow Bug | 工作流程问题 |
| Architecture Bug | 架构设计问题 |
| Regression | 已知的回归问题 |

---

### Regression Detection Types

| Type | Description |
|------|-------------|
| New Issue | 新发现的问题 |
| Known Regression | 已知的回归问题 |
| Requirement Change | 需求变更导致 |
| Architecture Defect | 架构缺陷 |

---

### Forbidden Behaviors List

| Behavior | Description |
|----------|-------------|
| 重构 | 修改代码结构 |
| 顺手优化 | 不相关的性能优化 |
| 架构改进 | 修改架构设计 |
| 改动无关模块 | 修改与问题无关的文件 |
| 添加新功能 | 新增功能代码 |
| 修改代码风格 | 格式化/风格调整 |
| 重命名变量/函数 | 命名变更 |
| 添加新文件 | 创建新文件 |
| 删除现有文件 | 删除已有文件 |

---

### Stop Conditions List

| Condition | Description |
|-----------|-------------|
| 需要修改 Forbidden Files | 发现需要修改禁止的文件 |
| 问题无法在 Allowed 内解决 | 边界内无法解决当前问题 |
| 发现新 regression 风险 | 发现新的潜在回归风险 |
| 发现需求理解偏差 | 发现对需求理解有误 |
| 需要扩大修改范围 | 发现需要修改更多文件 |
| 需要新增文件 | 发现需要创建新文件 |
| 需要修改测试文件外的其他文件 | 超出预期范围 |

---

### Core Principles

| Principle | Description |
|-----------|-------------|
| Analyze First, Patch Later | 先分析，后修改 |
| Patch Boundary > Prompt Engineering | 边界约束比提示技巧更重要 |
| Regression Rules > Long Context | 规则沉淀比长上下文更重要 |
| 限制 AI 的思考范围，比增强 AI 更重要 | 约束比能力更重要 |

---

### Design Decisions Log

| Decision | Choice | Reason |
|----------|--------|--------|
| Skill 形态 | 通用 Prompt 规范 | 跨 AI 工具兼容（Claude/Cursor/Copilot） |
| AI 接口 | 无内置 AI | 简化实现，用户在各自 AI 工具中使用 |
| 规则存储 | 项目级 | 每个项目独立，互不影响 |
| 辅助工具 | v2 规划 | 第一版聚焦核心流程，收敛范围 |
| 长期规则写入 | 用户明确要求 | 避免规则膨胀，只有重大 regression 才沉淀 |
| LOCKED_PLAN | 必须 | 防止 Implement 阶段变轨 |
| Verify 阶段 | 必须 | 确保 AI 执行符合边界 |

---

### v2 Planning (Not in v1)

| Feature | Description | Priority |
|---------|-------------|----------|
| `tools/analyze.sh` | grep/symbol 搜索脚本 | High |
| `tools/git-context.sh` | Git diff/blame 提取 | High |
| `tools/ast-parse.py` | AST 解析器（Python/JS/TS） | Medium |
| `tools/dep-graph.py` | 依赖图生成器 | Medium |
| 自动边界检测 | 基于工具分析自动生成 BOUNDARY.md | Low |
| RR 规则检索 | 自动检索相关 RR Rules | Low |

---

### Testing Checklist

| Item | Status |
|------|--------|
| 模板格式符合需求文档规范 | ✅ |
| .rr-example 包含完整工作流文件 | ✅ |
| RR_SKILL.md 定义四阶段流程 | ✅ |
| LOCKED_PLAN 机制完整描述 | ✅ |
| BOUNDARY.md 包含四个核心部分 | ✅ |
| VERIFY_REPORT.md 包含三状态判定 | ✅ |
| Trigger Keywords 定义清晰 | ✅ |
| Emergency Stop 定义清晰 | ✅ |

---

### Known Limitations (v1)

1. 无辅助工具支持，完全依赖 AI 主观判断边界
2. 无法自动检测是否超出 Allowed Files
3. 无法自动检测是否触碰 Forbidden Behaviors
4. Verify 阶段依赖 AI 自我报告，无客观验证
5. 长期规则需手动维护，无自动索引

---

### How to Upgrade to v1

在项目中使用：

```bash
# 1. 创建目录
mkdir -p .rr/rules .rr/current

# 2. 复制模板（可选，AI 也会自动生成）
cp templates/*.md .rr/current/

# 3. 在 AI 工具中加载 RR_SKILL.md
# Claude Code: 添加到 CLAUDE.md
# Cursor: 添加到 .cursorrules
# Copilot: 对话中发送 RR_SKILL.md 内容

# 4. 开始使用
# 输入: RR 分析：[问题描述]
```

---

### Contributors

- Requirements: RR_Skill_Requirement.md
- Implementation: Claude (glm-5)
- Date: 2024-05-24