# PatchGuard - AI Patch Governance Protocol

治理 AI Coding 中的增量修改行为，防止 patch explosion、scope creep、architecture drift。

---

## Project Overview

PatchGuard 是一个 **AI Patch Governance 协议**，用于治理 AI Coding 工具的代码修改行为。

核心目标：
- 不乱改
- 不扩大问题范围
- 不破坏已有系统
- 不发生上下文漂移
- 不发生架构侵蚀
- **确认原始问题是否被修复**

---

## Why PatchGuard

AI Coding 工具在修改代码时存在以下问题：

| Problem | Description |
|---------|-------------|
| 疯狂涂改 | AI 小修 bug 时不停改代码，越改越多 |
| 范围扩大 | 改了一个地方，顺手改其他地方 |
| 顺手重构 | 局部修复时顺便重构、优化 |
| 重新解释需求 | 开发阶段改变对问题的理解 |
| 只完成流程 | 填模板但没有证明问题修复 |

PatchGuard 通过结构化流程解决这些问题。

---

## What PatchGuard Is Not

明确边界：

- **不是代码生成器** - 不生成代码
- **不是自动修复工具** - 不自动修复 bug
- **不是自动测试系统** - 不自动测试代码
- **不是 multi-agent harness** - 不使用多 agent 架构
- **V2-alpha 是 Prompt Protocol** - 不是自动验证工具，仍需人工审查

---

## Core Workflow

```
rr analyze → human correction → locked plan → rr commit → rr implement → rr verify
```

| Phase | Trigger | Purpose |
|-------|---------|---------|
| Analyze | `RR 分析` | 分析问题，禁止改代码 |
| Commit | `确认计划` | 锁定边界，生成约束 |
| Implement | `开始执行` | 按约束修改代码 |
| Verify | `RR 验证` | 确认问题修复，检查边界合规 |

---

## Key Principles

| Principle | Description |
|-----------|-------------|
| Analyze First, Patch Later | 先分析，后修改 |
| Boundary Before Implementation | 先划定边界，再开始修改 |
| Locked Plan Means No Reinterpretation | 锁定后禁止重新解释需求 |
| Problem First, Process Second | 流程服务于问题修复，禁止填模板 |
| Minimum Sufficient Artifact | 小问题短写，复杂问题展开，禁止编造 |
| Evidence Over Claim | 需要证据证明问题修复，不是只填模板 |

---

## Quick Start

### 1. 在 AI 工具中加载 RR_SKILL.md

**Claude Code**: 添加到 `CLAUDE.md`
**Cursor**: 添加到 `.cursorrules`
**Copilot/其他**: 在对话中发送 `RR_SKILL.md` 内容

### 2. 创建项目目录

```bash
mkdir -p .rr/rules .rr/current .rr/state .rr/archive
```

### 3. 开始使用

输入触发词启动工作流：
```
RR 分析：[问题描述]
```

多轮校对后：
```
确认计划
开始执行
RR 验证
```

查看 VERIFY_REPORT 确认问题是否修复。

---

## Repository Structure

```
PatchGuard/
├── RR_SKILL.md              # 核心 Skill Prompt 规范
├── templates/               # 模板文件
│   ├── ANALYZE_REPORT.md
│   ├── BOUNDARY.md
│   ├── CURRENT_STATE.md
│   ├── LAST_LOCKED_PLAN.md
│   ├── PATCH_PLAN.md
│   ├── PATCH_RULES.md
│   ├── TASK_PACKET.md
│   └── VERIFY_REPORT.md
├── .rr-example/             # 完整工作流示例
│   ├── rules/               # 长期规则示例
│   ├── current/             # 当前 patch 示例
│   ├── state/               # 状态文件示例
│   └── archive/             # 归档规范示例
├── docs/
│   ├── QUICK_START.md       # 快速上手指南
│   └── WORKFLOW.md          # 详细工作流说明
├── VERSION.md               # 版本记录
└── RR_Skill_Requirement.md  # 历史需求文档（不是当前规则）
```

---

## Current Version: V2.0-alpha

**已支持**：
- current / rules / state / archive 目录结构
- Boundary / Locked Plan / Verify Report
- Problem Understanding / Fix Verification
- Minimum Sufficient Artifact 原则

**尚未支持**：
- tools/（辅助分析工具）
- gates/（自动检查）
- AST 解析
- git diff 自动检查
- multi-agent harness

**Future tool support should only be considered after the protocol proves useful in real patches.**

---

## Read Next

- [docs/QUICK_START.md](docs/QUICK_START.md) - 快速上手指南
- [docs/WORKFLOW.md](docs/WORKFLOW.md) - 详细工作流说明
- [RR_SKILL.md](RR_SKILL.md) - 核心 Skill Prompt 规范
- [VERSION.md](VERSION.md) - 版本记录和变更历史