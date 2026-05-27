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
| **跳过 Analyze** | AI 以"简单问题"为由绕过流程，直接修改代码 |

PatchGuard 通过结构化流程解决这些问题。

---

## Verify Review Dependency (v3.1)

Verify 阶段必须先检测代码审查依赖 skill：`mr-review` / `mr_review`。

- 优先使用当前运行环境已暴露的同名 skill
- 其次检查 PatchGuard 同级目录中的 `../mr_review` 或 `../mr-review`
- 缺失时必须下载/安装到可写、可复用的 skills root
- 安装目标不得写死为某个用户、某个机器或某个 agent 私有路径
- 若安装失败或未获授权，VERIFY_REPORT 必须记录缺口，最终状态不能为 PASS

---

## Critical Rule: Explicit Invocation

**只要用户显式调用 `/PatchGuard`，就必须进入流程。**

禁止以下 shortcut 理由：

| Forbidden Excuse | Why Forbidden |
|------------------|---------------|
| "问题很简单" | 简单问题也需要边界确认 |
| "只改一两行" | 一行代码也可能引入 regression |
| "根因很明确" | 明确根因不等于边界明确 |
| "可以快速修复" | 快速修复不等于安全修复 |

**RR-006**: Explicit Invocation Override - 禁止以任何理由跳过 Analyze。

---

## Activation Rule Check (v3.0)

### Activation Handshake

调用 `/PatchGuard` 后第一响应：

```
PatchGuard Activated
Phase: Analyze
Code Modification: Disabled
Patch ID: [Provided / Proposed / Pending]
Existing Patch Session: [None / Found]

Rules Check:
- Matching Promoted Rule: [Found / Missing / Unknown]
- Candidate Needed: [Yes / No]

Next Step: Problem Understanding
```

### Missing Rule Does NOT Block

**缺少对应 promoted rule 不阻塞 activation。**

即使 PATCH_RULES.md 完全不存在，`/PatchGuard` 也必须激活。

| Rules Status | Behavior |
|--------------|----------|
| Found | 引用规则，继续 |
| Missing | 标记缺失，继续 |
| Unknown | 标记未知，继续 |

### No Auto-Promotion

**AI 不能自动写入 PATCH_RULES.md。**

即使发现需要新规则：
- AI 只能生成 RR Candidate
- Human 必须确认 Promote
- 未确认前不能成为 active rule

---

## What PatchGuard Is Not

明确边界：

- **不是代码生成器** - 不生成代码
- **不是自动修复工具** - 不自动修复 bug
- **不是自动测试系统** - 不自动测试代码
- **不是 multi-agent harness** - 不使用多 agent 架构
- **V3-beta 是 Prompt Protocol** - 不是自动验证工具，仍需人工审查

---

## User Entry Point (v3.0)

### Single Entry: /PatchGuard

用户只需要输入一个命令：

```
/PatchGuard
```

后面跟需求或 bug 描述。

**不需要用户手动输入阶段命令。**

内部阶段自动线性推进，只在关键确认点暂停等待用户确认。

### Example

```
/PatchGuard

Patch ID: 2026-05-24-comment-replies

请实现评论追评 / 回复功能……
[完整需求]
```

### Automatic Flow

```
/PatchGuard + 需求
↓
Analyze (自动) → [暂停] 等待确认
↓
用户确认 → Lock (自动) → [暂停] 等待确认
↓
用户确认 "开始实现" → Implement (自动) → Verify (自动)
↓
[暂停] 输出 VERIFY_REPORT
↓
Optional: Promote RR Candidate
↓
归档
```

---

## Key Principles

| Principle | Description |
|-----------|-------------|
| Single Entry Point | 用户只输入 `/PatchGuard` |
| Automatic Progression | 内部阶段自动推进 |
| Pause at Confirmations | 只在关键点暂停等待用户确认 |
| One Patch One Directory | 每个 patch 独立 `.rr/patches/<patch-id>/` |
| Patch ID Must Confirm | AI 可以提议，但必须用户确认 |
| Locked Plan Cannot Change | Lock 后不得修改 Patch ID 或边界 |
| Problem First, Process Second | 流程服务于问题修复 |
| Human Controls Promote | AI 提议，Human 决定 |

---

## Quick Start

### 1. 在 AI 工具中加载 SKILL.md

**Claude Code**: 添加到 `CLAUDE.md` 或放入 `.claude/skills/` 目录
**Cursor**: 添加到 `.cursorrules`
**Copilot/其他**: 在对话中发送 `SKILL.md` 内容

### 2. 开始使用

输入命令启动工作流：
```
/PatchGuard

Patch ID: 2026-05-24-your-topic

[需求或 bug 描述]
```

AI 会自动：
- 进入 Analyze 阶段
- 提出或确认 Patch ID
- 创建 `.rr/patches/<patch-id>/` 目录
- 生成 ANALYZE_REPORT.md

### 3. 确认点介入

只在以下确认点介入：

| Confirmation Point | User Input |
|--------------------|------------|
| Analyze Result | `确认` / `修改分析` / `缩小边界` |
| Locked Plan | `开始实现` / `修改分析` |
| Verify Result | 查看 VERIFY_REPORT，决定是否提交 |
| RR Candidate | `Promote` / `Reject` / `Defer` |

**不需要手动触发阶段命令。**

---

## Directory Structure (v3.0)

每个 patch 拥有独立目录：

```
.rr/
├── patches/
│   ├── <patch-id>/
│   │   ├── ANALYZE_REPORT.md
│   │   ├── BOUNDARY.md
│   │   ├── PATCH_PLAN.md
│   │   ├── TASK_PACKET.md
│   │   ├── VERIFY_REPORT.md
│   │   ├── CURRENT_STATE.md
│   │   └── LAST_LOCKED_PLAN.md
│   │
│   └── <another-patch-id>/    # 多 patch 并存
│       └── ...
│
├── rules/
│   ├── PATCH_RULES.md         # 全局长期规则
│   └── CORE_MODULES.md
│
└── archive/
    └── <patch-id>/            # 已完成 patch 归档
```

**Key Changes from v2.0**:

| v2.0 | v3.0 |
|------|------|
| `.rr/current/` 全局唯一 | `.rr/patches/<patch-id>/` 每个 patch 独立 |
| `.rr/state/` 全局唯一 | 状态文件移入 patch 目录 |
| 单 patch session | 多 patch session 并存 |

---

## Repository Structure

```
PatchGuard/
├── SKILL.md                  # 核心 Skill Prompt 规范
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
│   ├── patches/             # patch session 示例
│   │   └── sample-comment-replies/
│   ├── rules/               # 长期规则示例
│   └── archive/             # 归档规范示例
├── docs/
│   ├── QUICK_START.md       # 快速上手指南
│   └── WORKFLOW.md          # 详细工作流说明
├── VERSION.md               # 版本记录
└── RR_Skill_Requirement.md  # 历史需求文档（不是当前规则）
```

---

## Current Version: V3.0-beta

**已支持**：
- 单入口 `/PatchGuard`
- 自动阶段推进
- Patch ID 确认机制
- `.rr/patches/<patch-id>/` 独立目录
- 多 patch session 并存
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
- [SKILL.md](SKILL.md) - 核心 Skill Prompt 规范
- [VERSION.md](VERSION.md) - 版本记录和变更历史
