# PatchGuard Version Log

## Historical Records Notice

**历史版本只作为 changelog，不作为当前行为规则。**

当前行为规则只以最新版本和 `SKILL.md` 为准。

历史版本中的状态定义、表述可能已过时，请参考最新版本。

---

## v3.0.1 (2024-05-24): Explicit Invocation Override

### Release Summary

修复显式调用优先级规则，防止 AI 以 "简单问题" 为由绕过 PatchGuard。

**核心问题**：用户显式调用 `/PatchGuard` 后，AI 仍然以 "问题简单" "只改两行" 等理由跳过 Analyze，直接修改代码。

**解决方案**：增加最高优先级规则 Explicit Invocation Override。

---

### Key Changes

#### 1. Explicit Invocation Override

**只要用户消息中显式出现 `/PatchGuard`，就必须进入流程。**

这是最高优先级规则，高于：
- 问题复杂度判断
- AI 自行判断
- 任何 shortcut 理由

#### 2. Forbidden Bypass Reasons

禁止使用以下理由绕过 PatchGuard：

| Forbidden Reason | Why Forbidden |
|------------------|---------------|
| "问题很简单" | 简单问题也需要边界确认 |
| "只改一两行" | 一行代码也可能引入 regression |
| "根因很明确" | 明确根因不等于边界明确 |
| "可以快速修复" | 快速修复不等于安全修复 |
| "上一个 patch 已 PASS" | 新问题是新 patch |

#### 3. Required First Response

如果用户显式调用 `/PatchGuard`：

1. 必须进入 Analyze
2. 禁止修改任何代码
3. 确认或提议 Patch ID
4. 处理未归档 patch

#### 4. Unarchived Patch Handling

检测到未归档 patch 时，必须询问用户选择：

- Continue existing patch
- Reopen existing patch
- Create new patch
- Archive existing patch first

AI 不得自行判断新消息属于哪个 patch。

#### 5. Simple Problems Must Also Analyze

即使修复只需要一行代码，也必须输出最小 Analyze：

- Problem Understanding
- Current Wrong Behavior
- Expected Behavior
- Patch ID
- Boundary
- Do Not Touch
- Minimal Patch Plan

#### 6. RR-006: Explicit Invocation Override

将 shortcut 行为沉淀为 regression pattern (RR-006)，写入 `.rr-example/rules/PATCH_RULES.md`。

---

### File Changes

| File | Change |
|------|--------|
| SKILL.md | 新增 Explicit Invocation Override 章节、Unarchived Patch Handling 章节 |
| README.md | 新增 Critical Rule: Explicit Invocation 章节 |
| docs/QUICK_START.md | 新增 Critical Rule: Explicit Invocation 章节、Unarchived Patch Handling |
| docs/WORKFLOW.md | 新增 Critical Rule: Explicit Invocation Override 章节 |
| templates/PATCH_RULES.md | 新增 Critical Rule: Explicit Invocation 定义 |
| .rr-example/rules/PATCH_RULES.md | 新增 RR-006: Explicit Invocation Override |
| VERSION.md | 新增 v3.0.1 记录 |

---

### Regression Pattern (RR-006)

| RR | Module | Severity | Summary |
|----|--------|----------|---------|
| RR-006 | explicit invocation | **Critical** | 禁止以任何理由跳过 Analyze |

典型违规：
- "root cause is clear"
- "only two lines need changing"
- "I can quickly fix this"
- "previous patch was already PASS"

---

## v3.0-beta (2024-05-24): Interaction Entry & Patch Session Directory Upgrade

### Release Summary

用户入口与 Patch Session Directory 结构升级。

**核心变化**：
- 用户-facing 命令改为单入口 `/PatchGuard`
- analyze / lock / implement / verify / promote 改为内部阶段，自动推进
- 新增 Patch ID 确认机制
- 将 `.rr/current/` 和 `.rr/state/` 重构为 `.rr/patches/<patch-id>/`
- CURRENT_STATE.md 和 LAST_LOCKED_PLAN.md 迁移到 patch session 目录
- .rr/rules/ 保持全局长期规则
- .rr/archive/<patch-id>/ 作为完成 patch 归档

---

### Key Changes

#### 1. Single Entry Point

**Before (v2.0)**:
```
RR 分析：[问题描述]
确认计划
开始执行
RR 验证
```

**After (v3.0)**:
```
/PatchGuard

Patch ID: 2026-05-24-comment-replies

[需求描述]
```

用户只输入一个命令，内部阶段自动推进，只在关键确认点暂停。

#### 2. Internal Phases (Not User Commands)

| Phase | v2.0 | v3.0 |
|-------|------|------|
| Analyze | 用户输入 `RR 分析` | 自动进入 |
| Lock | 用户输入 `确认计划` | 用户确认 `确认` 后自动进入 |
| Implement | 用户输入 `开始执行` | 用户确认 `开始实现` 后自动进入 |
| Verify | 用户输入 `RR 验证` | Implement 完成后自动进入 |

#### 3. Patch ID Mechanism

新增规则：
- 用户可以显式提供 Patch ID
- 如果用户没有提供，AI 可以提议 `Proposed Patch ID`
- Proposed Patch ID 必须等待用户确认
- 用户确认前不得创建 `.rr/patches/<patch-id>/` 目录
- Lock 后不得修改 Patch ID

#### 4. Directory Structure

**Before (v2.0)**:
```
.rr/
├── current/         # 全局唯一
├── state/           # 全局唯一
├── rules/
└── archive/
```

**After (v3.0)**:
```
.rr/
├── patches/
│   └── <patch-id>/  # 每个 patch 独立目录
│       ├── ANALYZE_REPORT.md
│       ├── BOUNDARY.md
│       ├── PATCH_PLAN.md
│       ├── TASK_PACKET.md
│       ├── VERIFY_REPORT.md
│       ├── CURRENT_STATE.md
│       └── LAST_LOCKED_PLAN.md
│
├── rules/
│   ├── PATCH_RULES.md
│   └── CORE_MODULES.md
│
└── archive/
    └── <patch-id>/
```

**Key Changes**:
- ❌ 不再使用 `.rr/current/` 作为全局唯一目录
- ❌ 不再使用 `.rr/state/` 作为全局唯一目录
- ✅ 每个 patch 拥有独立 `.rr/patches/<patch-id>/` 目录
- ✅ 多 patch session 可并存

#### 5. Pause Points

| Pause Point | Wait For |
|-------------|----------|
| Analyze Result | Problem Understanding + Patch ID 确认 |
| Locked Plan | 是否开始实现确认 |
| Verify Result | 查看 VERIFY_REPORT |
| RR Candidate | Promote / Reject / Defer 决定 |

---

### File Changes

| File | Change |
|------|--------|
| SKILL.md | 完全重写，单入口 `/PatchGuard`，自动阶段推进，Patch ID 机制，新目录结构 |
| README.md | 更新为单入口说明，新目录结构，移除手动阶段命令 |
| docs/QUICK_START.md | 更新为 `/PatchGuard` 入口，Patch ID 规则，新目录结构 |
| docs/WORKFLOW.md | 更新为内部阶段说明，Pause Points，Patch ID 规则 |
| templates/ANALYZE_REPORT.md | 增加 Patch ID section |
| templates/CURRENT_STATE.md | 增加 patch_id 字段，更新 Phase Flow |
| templates/LAST_LOCKED_PLAN.md | 增加 Patch ID section |
| .rr-example/current/ | 迁移到 .rr-example/patches/sample-comment-replies/ |
| .rr-example/state/ | 迁移到 .rr-example/patches/sample-comment-replies/ |
| .rr-example/rules/CORE_MODULES.md | 新增，核心模块列表示例 |
| .rr-example/patches/README.md | 新增，patch session 结构说明 |

---

### Path Replacements

所有 `.rr/current/` 和 `.rr/state/` 引用已替换为 `.rr/patches/<patch-id>/`

---

### Not Implemented

以下功能未在 v3.0 实现：
- tools/（辅助分析工具）
- gates/（自动检查）
- AST 解析
- git diff 自动检查
- multi-agent harness
- 自动化脚本
- Light / Full 模式

---

## v2.0-alpha.5 (2024-05-24): Skill File Rename

### Release Summary

重命名 `RR_SKILL.md` → `SKILL.md`，符合 Claude Code skills 目录命名规范。

---

### File Changes

| File | Change |
|------|--------|
| RR_SKILL.md → SKILL.md | 重命名，Claude Code 查找 SKILL.md |
| README.md | 更新引用 RR_SKILL.md → SKILL.md |
| VERSION.md | 更新引用 RR_SKILL.md → SKILL.md |
| RR_Skill_Requirement.md | 更新历史文档中的引用 |

---

## v2.0-alpha.4 (2024-05-24): Promote Criteria Clarification

### Release Summary

明确 Promote Criteria 语义：Human Confirmation 是唯一强制条件，其他 criteria 是 evidence。

**核心修正**：
- Human explicitly confirms Promote 是唯一 mandatory
- Major Regression / Typical Error Pattern / Core Module 是 evidence，不是全部必选
- AI cannot auto-promote based on criteria alone
- 至少一项 evidence 匹配即可 propose，但必须 Human Promote 才能成为 active rule

---

### Fixes

#### Promote Criteria Clarification

**Before**（误导性表述）：
```
只有符合以下条件才 promote：
- Major Regression: Yes
- Typical Error Pattern: Yes
- Core Module: Yes
- Human Confirmation: Required
```
→ 容易被理解为三项全部 Yes 才能 Promote

**After**（正确表述）：
```
Promotion requires:
- Human explicitly confirms Promote

And at least one of the following evidence criteria:
- Major Regression
- Typical Error Pattern
- Core Module confirmed by human
- Prevents future scope creep or patch explosion
- Repeated error observed in previous patches

Do not require all evidence criteria to be Yes.
```

#### Promotion Checklist Fix

**Before**：所有 criteria 混在一起，无 mandatory vs evidence 区分

**After**：
```
Mandatory:
- Human Decision = Promote (Required)

Evidence (at least one should match):
- Has this error happened before?
- Did it cause major regression?
- Is it likely to recur?
- ...

Blockers (any Yes = do not promote):
- Does it duplicate existing rule?
- Is it too narrow?

Key Rule: If Human Decision ≠ Promote, even with strong evidence, cannot become active.
```

#### templates/PATCH_RULES.md Fix

**Before**：
```
只有符合以下条件才写入：
1. 重大 regression
2. 典型错误模式
3. 核心模块
```

**After**：
```
AI may use the following criteria to propose an RR Candidate.
These criteria are evidence for promotion, not automatic promotion rules.
Only human-confirmed candidates can become active PATCH_RULES.
```

---

### File Changes

| File | Change |
|------|--------|
| SKILL.md Promote Criteria | Human Confirmation 是唯一 mandatory，其他是 evidence |
| docs/WORKFLOW.md Promote Criteria | 同步语义修正 |
| templates/PATCH_RULES.md Important | 修正 "只有符合以下条件才写入" 表述 |
| templates/PATCH_RULES.md Promotion Checklist | 区分 Mandatory / Evidence / Blockers |
| VERSION.md | v2.0-alpha.4 记录 |

---

## v2.0-alpha.3 补充 (2024-05-24): PATCH_RULES Promote Governance

### Release Summary

PATCH_RULES promote 机制治理修正，明确 AI 不能自行 promote。

**核心原则**：
- AI may propose RR Candidates, but only human-confirmed rules can be promoted to PATCH_RULES.

---

### Fixes

#### Core Principle Added

**AI 只能生成 RR Candidate，不能自行 promote 到 PATCH_RULES。**

只有 Human 明确确认 Promote 后，规则才能进入长期 PATCH_RULES。

#### Definition Added

**Major Regression**：
- Data loss
- Incorrect database write
- Broken save pipeline
- Broken render pipeline for core page
- Authentication / permission / payment / order flow failure
- Irreversible state corruption
- Production-blocking error

**Typical Error Pattern**：
- Template changed but JS state not updated
- Update path fixed but insert path missed
- Render fixed but save pipeline missed
- Create/copy mode missed
- Local bug fixed by unrelated refactor
- Patch scope expanded beyond locked boundary
- Verification claims fixed but no evidence was provided

**Core Module**：
- AI must not assume a module is core
- A module is core only if listed in `.rr/rules/CORE_MODULES.md` or confirmed by human

#### RR Candidate Template Added

每个 Candidate 必须包含：
- Candidate ID
- Source Patch
- Original Problem
- Proposed Rule
- Evidence
- Promotion Criteria Matched
- Duplicate Check
- Human Decision: Promote / Reject / Defer
- Decision Reason

#### Promotion Checklist Added

| Criterion | Yes/No | Evidence |
|-----------|--------|----------|
| Has this error happened before? | | |
| Did it cause data loss, crash, or core feature failure? | | |
| Is it likely to recur in similar patches? | | |
| Is the affected module explicitly marked as core? | | |
| Would this rule prevent future scope creep or patch explosion? | | |
| Is the rule specific and actionable? | | |
| Does it duplicate an existing rule? | | |
| Is it too narrow or one-off? | | |

#### Do Not Promote Added

- The bug is one-off
- The rule is vague
- The rule only describes this exact patch
- The rule cannot guide future patches
- The rule duplicates an existing rule
- There is no human confirmation
- The only evidence is AI speculation

---

### File Changes

| File | Change |
|------|--------|
| templates/PATCH_RULES.md | 增加核心原则、定义、RR Candidate 模板、Promotion Checklist、Do Not Promote |
| SKILL.md Promote Rule | 明确 AI 不能自行 promote |
| docs/WORKFLOW.md Promote Rule | 明确 AI 只能生成 Candidate，Human 决策 |
| .rr-example/rules/PATCH_RULES.md | 区分 Promoted Rules 和 RR Candidates |

---

## v2.0-alpha.3 (2024-05-24)

### Release Summary

Single Source of Truth 修正，消除 SKILL.md、templates/、docs/WORKFLOW.md 之间的模板漂移。

**核心变更**：
- templates/ 作为报告结构单一来源
- SKILL.md 只保留行为规则，不重复内嵌完整模板
- WORKFLOW.md 同步 v2.0 Output 结构

---

### Fixes

#### Template Drift Fix

SKILL.md 与 templates/ 存在重复定义和不一致：

**Before**：
- SKILL.md 内嵌完整旧模板（v1.0/v1.1 结构）
- templates/ 已升级到 v2.0 Problem-Focused
- docs/WORKFLOW.md 仍写 v1.1 结构

**After**：
- templates/ 作为报告结构单一来源
- SKILL.md 只保留行为规则 + Output Structure 表格
- WORKFLOW.md 同步 v2.0 结构

#### Phase 1 Output Structure (v2.0)

| Section | Required | Priority |
|---------|----------|----------|
| Problem Understanding | ✅ | **First** |
| Problem Classification | ✅ | Second |
| Impact Radius | ✅ | Third |
| Regression Detection | ✅ | Fourth |
| Risk Analysis | ✅ | Fifth |
| Minimal Fix Strategy | ✅ | Sixth |
| Version History | ✅ | Last |

**硬规则**：Problem Understanding 未填写清楚，禁止进入 Problem Classification。

#### Phase 4 Output Structure (v2.0)

| Section | Required | Priority |
|---------|----------|----------|
| Fix Verification | ✅ | **First** |
| Modified Files | ✅ | Second |
| New Files Created | ✅ | Third |
| Boundary Check | ✅ | Fourth |
| Unverified Items | ✅ | Fifth |
| Evidence | ✅ | Sixth |
| Manual Verification Required | ✅ | Seventh |
| Risks | ✅ | Eighth |
| Final Recommendation | ✅ | Last |

**硬规则**：Fix Result 不是 Fixed，VERIFY_REPORT 不能是 PASS。

---

### File Changes

| File | Change |
|------|--------|
| SKILL.md Phase 1 | 移除内嵌模板，改为 Output Structure 表格 + Problem Understanding 优先规则 |
| SKILL.md Phase 4 | 移除内嵌模板，改为 Output Structure 表格 + Fix Verification 优先规则 |
| docs/WORKFLOW.md Phase 1 | Output 表格更新为 v2.0，增加 Problem Understanding First |
| docs/WORKFLOW.md Phase 4 | Output 表格从 v1.1 更新为 v2.0，增加 Fix Verification First |
| docs/QUICK_START.md | 增加 Problem-Focused Governance 说明 |
| VERSION.md | v2.0-alpha.3 记录 |

---

## v2.0-alpha.2 (2024-05-24)

### Release Summary

Problem-Focused Governance 修正，让 PatchGuard 不只检查流程合规，还要检查原始问题是否被修复。

**新增原则**：
- Problem First, Process Second
- Minimum Sufficient Artifact

**新增章节**：
- Problem Understanding（Analyze 阶段）
- Fix Verification（Verify 阶段）

---

### New Principles

#### Problem First, Process Second

PatchGuard 的所有流程都必须服务于确认原始问题是否被修复。

禁止为了填模板而填模板。

流程合规不代表问题修复。

#### Minimum Sufficient Artifact

保持统一流程，不增加 Light / Full 模式。

**小问题可以短写**：
- 无关章节可以写 `N/A - not relevant to this patch`
- 禁止编造风险、编造影响范围、编造规则

**任何 patch 至少必须保留**：
- Problem
- Expected Behavior
- Current Wrong Behavior
- Boundary
- Do Not Touch
- Minimal Patch Plan
- Fix Verification

---

### New Sections

#### Problem Understanding (Analyze)

Analyze 阶段必须先确认问题本身：
- User Report
- Current Wrong Behavior
- Expected Behavior
- Confirmation Needed

如果未填写清楚，禁止进入 Problem Classification。

#### Fix Verification (Verify)

Verify 阶段必须先确认原始问题是否被修复：
- Original Problem
- Expected Behavior
- Actual Behavior After Patch
- Verification Method
- Fix Result: Fixed / Not Fixed / Partially Fixed / Unknown

如果 Fix Result 不是 Fixed，VERIFY_REPORT 不能是 PASS。

---

### File Changes

| File | Change |
|------|--------|
| SKILL.md | 增加 Core Principle、Minimum Sufficient Artifact、修正 WARNING 表述 |
| templates/ANALYZE_REPORT.md | 增加 Problem Understanding 部分 |
| templates/VERIFY_REPORT.md | 增加 Fix Verification 部分 |
| .rr-example/current/ANALYZE_REPORT.md | 增加 Problem Understanding 示例 |
| .rr-example/current/VERIFY_REPORT.md | 增加 Fix Verification 示例 |
| README.md | 重写为项目入口文档 |
| RR_Skill_Requirement.md | 标记为历史文档 |
| VERSION.md | v2.0-alpha.2 记录 |

---

### README.md Rewrite

重写顶层 README.md 为项目入口文档：

- Project Overview
- Why PatchGuard
- What PatchGuard Is Not
- Core Workflow
- Key Principles
- Quick Start
- Repository Structure
- Current Version（明确已支持/尚未支持）
- Read Next

移除 v2 Planning 中诱导进入 tools/AST 的表述。

改为：**Future tool support should only be considered after the protocol proves useful in real patches.**

---

## v2.0-alpha.1 (2024-05-24)

### Release Summary

V2-alpha 文案一致性修正，不增加新功能。

**修正内容**：
- 统一 PASS / WARNING / FAIL 的当前语义
- 移除"PASS 可提交"的误导表达
- 移除"WARNING 验证后必然可提交"的误导表达
- 将"FAIL 必须回滚"改为"FAIL 必须停止，并根据风险决定回滚或返回 rr analyze"

---

### Fixes

#### Status Definition 统一 (v2.0)

| Status | Action (v2.0) |
|--------|---------------|
| PASS | 边界检查通过，可进入提交前人工审查 |
| WARNING | 必须人工验证 Unverified Items 后，再决定是否进入提交前人工审查 |
| FAIL | 必须停止，并根据风险决定回滚或返回 rr analyze |

**消除表述**：
- ❌ "PASS 可提交"
- ❌ "PASS 边界检查通过，可提交"
- ❌ "WARNING 必须人工验证后才能提交"
- ❌ "WARNING 验证后提交"
- ❌ "FAIL 必须回滚"
- ❌ "FAIL 必须回滚并返回 Phase 1"

**替换表述**：
- ✅ "PASS：边界检查通过，可进入提交前人工审查"
- ✅ "WARNING：必须人工验证 Unverified Items 后，再决定是否进入提交前人工审查"
- ✅ "FAIL：必须停止，并根据风险决定回滚或返回 rr analyze"

---

### File Changes

| File | Change |
|------|--------|
| templates/VERIFY_REPORT.md | Status Definition 表述统一 |
| .rr-example/current/VERIFY_REPORT.md | 示例表述统一 |
| docs/WORKFLOW.md | Verify 阶段、Summary 表表述统一 |
| .rr-example/state/CURRENT_STATE.md | "人工验证后可提交"改为正确表述 |
| VERSION.md | 历史版本声明、v2.0-alpha.1 记录 |

---

## v2.0-alpha (2024-05-24)

### Release Summary

V2-alpha 最小切片：实现 Project State & Rule Memory。

**新增功能**：
- CURRENT_STATE.md - 记录当前 RR 阶段，防止阶段跳跃
- LAST_LOCKED_PLAN.md - 防止 implement 阶段重新解释需求
- archive 规范 - patch 完成后归档流程
- promote-rule 规则 - 明确不是每个 bug 都进入长期 PATCH_RULES
- 目录结构扩展：`.rr/state/`、`.rr/archive/`

---

### New Features

#### 1. CURRENT_STATE.md

记录当前 RR 流程的阶段状态。

**Phase Flow**：
```
none → analyze (DRAFT) → analyze (LOCKED) → commit → implement → verify → done
```

**硬规则**：
- 禁止跳跃阶段
- 每次阶段变化必须更新此文件

#### 2. LAST_LOCKED_PLAN.md

保存最近一次锁定的 Analyze 结果。

**硬规则**：
- Implement 阶段禁止重新解释需求
- 如需改变理解，必须 STOP 并返回 rr analyze

#### 3. Archive 规范

patch 完成后归档流程：
1. 确认 VERIFY_REPORT 状态为 PASS 或 WARNING（已人工验证）
2. 创建归档目录：`.rr/archive/YYYY-MM-DD-patch-XXX/`
3. 移动 `.rr/current/` 和 `.rr/state/` 文件到归档目录
4. 清空 `.rr/current/` 和 `.rr/state/`（准备下一个 patch）
5. 评估是否 promote rule

#### 4. Promote Rule 规则

只有符合以下条件才写入 `.rr/rules/PATCH_RULES.md`：

| Criterion | Required |
|-----------|----------|
| Severity | High / Critical |
| Pattern | Yes |
| Recurrence Risk | High |
| User Confirmation | Required |

**禁止**：将一次性 bug、低风险修复、简单 typo 写入长期规则。

---

### Directory Structure (v2.0)

```
.rr/
├── rules/          # 长期规则（跨 patch 复用）
├── current/        # 当前 patch 文档
├── state/          # 状态文件（防止跳跃/重新解释）
└── archive/        # 归档文件
```

| Directory | Purpose | Lifecycle |
|-----------|---------|-----------|
| `.rr/rules/` | 长期 Regression Rules | 永久 |
| `.rr/current/` | 当前 patch 文档 | 当前 patch |
| `.rr/state/` | RR 阶段状态 | 当前 patch |
| `.rr/archive/` | 已完成 patch 归档 | 永久 |

---

### Documentation Unification

统一 PASS/WARNING/FAIL 表述：

| Status | Action (v2.0) |
|--------|---------------|
| PASS | 边界检查通过，可进入提交前人工审查 |
| WARNING | 必须人工验证后才能决定是否提交 |
| FAIL | 必须停止并返回 rr analyze |

**消除表述**：
- ❌ "PASS 可提交"
- ❌ "WARNING 需补充验证后提交"
- ❌ "FAIL 必须回滚"

---

### File Changes

| File | Status | Change |
|------|--------|--------|
| templates/CURRENT_STATE.md | New | RR 阶段状态模板 |
| templates/LAST_LOCKED_PLAN.md | New | 锁定计划模板 |
| templates/PATCH_RULES.md | New | 长期规则模板（含 promote 规则） |
| .rr-example/state/CURRENT_STATE.md | New | 示例状态文件 |
| .rr-example/state/LAST_LOCKED_PLAN.md | New | 示例锁定计划 |
| .rr-example/archive/README.md | New | 归档规范说明 |
| SKILL.md | Modified | 增加 state/rules/archive 说明 |
| docs/QUICK_START.md | Modified | 增加目录说明、统一表述 |
| docs/WORKFLOW.md | Modified | 增加状态文件说明、promote 规则 |
| VERSION.md | Modified | v2.0-alpha 记录 |

---

### Not Implemented (v2 scope)

以下功能未在 v2.0-alpha 实现：

- tools/（辅助分析工具）
- gates/（自动检查 gates）
- AST 解析
- git diff 自动检查
- multi-agent harness
- 自动化脚本
- 自动判断代码是否越界

---

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

在 SKILL.md、QUICK_START.md、WORKFLOW.md 中明确说明：
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
| SKILL.md | Phase 4 状态定义修复、BOUNDARY.md Format 更新、Hard Rules 增加 |
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
├── SKILL.md                    # 核心 Skill Prompt 规范
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
| SKILL.md 定义四阶段流程 | ✅ |
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

# 3. 在 AI 工具中加载 SKILL.md
# Claude Code: 添加到 CLAUDE.md
# Cursor: 添加到 .cursorrules
# Copilot: 对话中发送 SKILL.md 内容

# 4. 开始使用
# 输入: RR 分析：[问题描述]
```

---

### Contributors

- Requirements: RR_Skill_Requirement.md
- Implementation: Claude (glm-5)
- Date: 2024-05-24