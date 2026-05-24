# RR Skill - Quick Start Guide

## What is RR Skill?

RR Skill 是一个 **AI Patch Governance System**，用于治理 AI Coding 工具的代码修改行为。

**核心目标**：让 AI 在修改代码时：
- 不乱改
- 不扩大问题范围
- 不破坏已有系统
- 不发生上下文漂移
- 不发生架构侵蚀

---

## Important: V2 Limitations

**V2-alpha 是 Prompt Protocol，不是自动验证工具。**

- `rr verify` 输出依赖 AI 自查，**仍然需要人工审查**
- **PASS 不代表业务完全正确**，只代表边界检查通过
- **WARNING 必须人工验证 Unverified Items 后，再决定是否进入提交前人工审查**
- Unverified Items 存在时，状态不能是 PASS

---

## Status Definition (v2.0)

| Status | Condition | Meaning |
|--------|-----------|---------|
| PASS | 无越界、无 Forbidden、无未验证关键项 | 边界检查通过，可进入提交前人工审查 |
| WARNING | 无越界、无 Forbidden，但存在未验证项 | **必须人工验证 Unverified Items 后，再决定是否进入提交前人工审查** |
| FAIL | 触碰 Forbidden、超出 Allowed、违反 Locked Plan | 必须停止，并根据风险决定回滚或返回 rr analyze |

---

## Quick Setup

### Step 1: 在项目中创建 .rr 目录

```bash
mkdir -p .rr/rules .rr/current .rr/state .rr/archive
```

### Step 2: 复制模板文件

将 `templates/` 目录下的文件复制到 `.rr/current/`：

```bash
cp templates/ANALYZE_REPORT.md .rr/current/
cp templates/BOUNDARY.md .rr/current/
cp templates/PATCH_PLAN.md .rr/current/
cp templates/TASK_PACKET.md .rr/current/
cp templates/VERIFY_REPORT.md .rr/current/
```

### Step 3: 在 AI 工具中加载 RR_SKILL.md

**Claude Code**:
```
将 RR_SKILL.md 内容添加到项目的 CLAUDE.md 或在对话中引用
```

**Cursor**:
```
将 RR_SKILL.md 内容添加到 .cursorrules 文件
```

**Copilot / 其他 AI**:
```
在对话开始时发送 RR_SKILL.md 内容，告知 AI 遵循此规范
```

---

## Basic Usage

### 发现问题 → RR Analyze

在 AI 工具中输入：

```
RR 分析：编辑产品页面时，variants 数据回显丢失
```

AI 会输出 ANALYZE_REPORT.md 内容。

---

### 多轮校对

查看 AI 输出的分析报告，如有问题进行纠正：

```
纠正：这个问题还涉及 ext_save_data.js 文件
```

AI 会迭代更新 ANALYZE_REPORT.md。

---

### 确认计划 → RR Commit

确认分析结果后，输入：

```
确认计划
```

AI 会生成：
- BOUNDARY.md（修改边界）
- PATCH_PLAN.md（修改计划）
- TASK_PACKET.md（任务包）

---

### 开始执行 → RR Implement

确认边界后，输入：

```
开始执行
```

AI 会严格按 TASK_PACKET.md 约束修改代码。

---

### 验证 → RR Verify

修改完成后，输入：

```
RR 验证
```

AI 会输出 VERIFY_REPORT.md，检查边界合规性。

---

## Emergency Stop

任何时候发现问题，输入：

```
停止，重新分析
```

AI 会立即停止，返回 Phase 1 重新分析。

---

## Directory Structure (v2.0)

```
project/
├── .rr/
│   ├── rules/
│   │   └── PATCH_RULES.md    # 长期规则（跨 patch 复用）
│   ├── current/              # 当前 patch 文件
│   │   ├── ANALYZE_REPORT.md
│   │   ├── BOUNDARY.md
│   │   ├── PATCH_PLAN.md
│   │   ├── TASK_PACKET.md
│   │   └── VERIFY_REPORT.md
│   ├── state/                # 状态文件
│   │   ├── CURRENT_STATE.md  # 当前 RR 阶段
│   │   └── LAST_LOCKED_PLAN.md # 最近锁定计划
│   └── archive/              # 归档文件
│       └── YYYY-MM-DD-patch-XXX/
```

### Directory Purpose

| Directory | Purpose | Lifecycle |
|-----------|---------|-----------|
| `.rr/rules/` | 长期 Regression Rules | 永久 |
| `.rr/current/` | 当前 patch 文档 | 当前 patch |
| `.rr/state/` | RR 阶段状态，防止跳跃 | 当前 patch |
| `.rr/archive/` | 已完成 patch 归档 | 永久 |

---

## Trigger Keywords Reference

| Trigger | Phase | Action |
|---------|-------|--------|
| `RR 分析` | Analyze | 分析问题，输出报告 |
| `确认计划` | Commit | 锁定计划，生成边界 |
| `开始执行` | Implement | 按约束修改代码 |
| `RR 验证` | Verify | 检查边界合规性 |
| `停止，重新分析` | Stop | 立即停止，返回 Analyze |

---

## Best Practices

1. **不要跳过 Analyze**：始终先分析，后修改
2. **多轮校对**：仔细检查分析结果，确保边界正确
3. **关注 Stop Conditions**：AI 报告停止条件时，认真对待
4. **WARNING 必须人工验证**：存在未验证项时必须人工确认
5. **PASS 仍需审查**：边界检查通过不代表业务完全正确
6. **新增文件需提前规划**：在 Analyze 阶段列入 Allowed New Files

---

## Example Session

查看 `.rr-example/` 目录获取完整工作流示例。