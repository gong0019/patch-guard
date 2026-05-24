# PatchGuard - RR Skill

**AI Patch Governance System** - 治理 AI Coding 工具的代码修改行为

---

## What is RR Skill?

RR Skill 是一个跨 AI 工具的通用 Skill 规范，用于约束 AI 的代码修改行为：

- **不乱改** - 强制先分析后修改
- **不扩大范围** - 明确 Allowed/Forbidden 边界
- **不破坏系统** - Regression Rules 防止再犯
- **不发生漂移** - LOCKED_PLAN 禁止变轨

---

## Quick Start

1. 在项目中创建 `.rr/` 目录
2. 在 AI 工具中加载 `RR_SKILL.md`
3. 使用触发词启动工作流

```bash
mkdir -p .rr/rules .rr/current
```

**触发词**：
- `RR 分析` → 分析问题
- `确认计划` → 锁定边界
- `开始执行` → 修改代码
- `RR 验证` → 检查合规

---

## Structure

```
PatchGuard/
├── RR_SKILL.md              # 核心 Skill 规范
├── templates/               # 模板文件
│   ├── ANALYZE_REPORT.md
│   ├── BOUNDARY.md
│   ├── PATCH_PLAN.md
│   ├── TASK_PACKET.md
│   └── VERIFY_REPORT.md
├── .rr-example/             # 完整工作流示例
│   ├── rules/
│   │   └── PATCH_RULES.md
│   └── current/
│       ├── ANALYZE_REPORT.md
│       ├── BOUNDARY.md
│       ├── PATCH_PLAN.md
│       ├── TASK_PACKET.md
│       └── VERIFY_REPORT.md
├── docs/
│   ├── QUICK_START.md
│   └── WORKFLOW.md
└── RR_Skill_Requirement.md  # 需求文档
```

---

## Four Phases

| Phase | Trigger | Purpose |
|-------|---------|---------|
| Analyze | `RR 分析` | 禁止改代码，先分析问题 |
| Commit | `确认计划` | 锁定边界，生成约束 |
| Implement | `开始执行` | 按约束修改代码 |
| Verify | `RR 验证` | 检查边界合规性 |

---

## Usage in Different AI Tools

**Claude Code**: 添加到 `CLAUDE.md`
**Cursor**: 添加到 `.cursorrules`
**Copilot/其他**: 在对话中发送 `RR_SKILL.md` 内容

---

## Documentation

- [QUICK_START.md](docs/QUICK_START.md) - 快速上手指南
- [WORKFLOW.md](docs/WORKFLOW.md) - 详细工作流说明

---

## Example

查看 `.rr-example/` 目录获取完整工作流示例。

---

## v2 Planning

- 辅助工具：grep/symbol 搜索、git diff/blame、AST 解析、依赖图生成