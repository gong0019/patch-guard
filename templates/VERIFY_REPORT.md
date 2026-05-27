# RR Verify Report

## Status

[PASS / WARNING / FAIL]

---

## Fix Verification (v2.0)

**Verify 阶段必须先确认原始问题是否被修复，再检查流程合规。**

### Original Problem

[从 ANALYZE_REPORT.md 的 Problem Understanding 部分引用]

### Expected Behavior

[从 ANALYZE_REPORT.md 的 Expected Behavior 引用]

### Actual Behavior After Patch

[修改后系统的实际行为是什么？]

### Verification Method

[如何验证问题已修复？测试方法？]

### Fix Result

| Result | Description |
|--------|-------------|
| Fixed | 问题已修复，Actual Behavior 匹配 Expected Behavior |
| Not Fixed | 问题未修复，Actual Behavior 仍不匹配 Expected Behavior |
| Partially Fixed | 问题部分修复，仍有残留问题 |
| Unknown | 无法确认是否修复，需要进一步验证 |

**如果 Fix Result 不是 Fixed，VERIFY_REPORT 不能是 PASS。**

---

## MR-Review Code Quality Assessment (v3.1 - Powered by mr-review)

**利用 `mr-review` 引擎对当前修改的代码进行全覆盖质量与安全性深度静态审查。**

### Dependency Resolution

- Skill name: [mr-review / mr_review]
- Resolved path: [已安装路径；缺失则填写 Missing]
- Install attempted: [Yes / No / Not Needed]
- Install source: [如有下载，填写来源 URL / installer / version]
- Required files checked: [SKILL.md, references/workflow.md, references/report-format.md]
- Review coverage: [列出已覆盖 changed files；若未全覆盖，状态不能为 PASS]

### Confirmed Bugs

[若有 Confirmed Bugs，请在此列出（影响文件、行号、根因、影响、修复方向），状态将强制为 FAIL]
- [必须填写：None 或问题列表，并附依据]

### Risks / Edge Cases

[列出可能存在的隐性运行时、发布或维护风险]
- [必须填写：None 或风险列表，并附依据]

### Verification Gaps

- **静态自查**：[说明已使用的 mr-review 文件、覆盖范围和结论]
- **动态局限性声明**：[说明本次无法运行的测试/环境限制；若已运行测试，列出命令和结果]

---

## Modified Files

| File | Changes Summary |
|------|-----------------|
| [file 1] | [修改内容摘要] |
| [file 2] | [修改内容摘要] |

---

## New Files Created

| File | Status |
|------|--------|
| [new file 1] | ✅ in Allowed Files / ❌ NOT in Allowed Files |

**Result**: [✅ all new files in Allowed / ❌ X new file(s) NOT in Allowed]

---

## Boundary Check

### Allowed Files Check

| File | Status |
|------|--------|
| [file 1] | ✅ in allowed |
| [file 2] | ✅ in allowed |
| [file 3] | ❌ NOT in allowed (if any) |

**Result**: [✅ all in allowed / ❌ X file(s) out of allowed]

---

### Forbidden Files Check

| File | Status |
|------|--------|
| [forbidden file 1] | ✅ not touched |
| [forbidden file 2] | ✅ not touched |
| [forbidden file X] | ❌ TOUCHED (if any) |

**Result**: [✅ none touched / ❌ touched: ...]

---

### Forbidden Behaviors Check

| Behavior | Status |
|----------|--------|
| 重构 | ✅ none / ❌ detected |
| 优化 | ✅ none / ❌ detected |
| 架构改动 | ✅ none / ❌ detected |
| 改动无关模块 | ✅ none / ❌ detected |
| 添加新业务功能 | ✅ none / ❌ detected |
| 修改代码风格 | ✅ none / ❌ detected |
| 重命名 | ✅ none / ❌ detected |

**Result**: [✅ none / ❌ detected: ...]

---

## Unverified Items

以下项目尚未验证：

- [ ] [未验证项 1]: [需要验证的内容]
- [ ] [未验证项 2]: [需要验证的内容]
- [ ] [未验证项 3]: [需要验证的内容]

**Unverified Items Count**: [数量]

---

## Evidence

修改证据记录：

| Evidence Type | Source | Details |
|---------------|--------|---------|
| Diff Source | [git diff / 手动记录] | [具体来源] |
| Modified Lines | [行数范围] | [修改行号] |
| Review Method | [AI 自查 / 人工审查] | [审查方式] |

---

## Manual Verification Required

以下项目需要人工验证：

| Item | Reason | Priority |
|------|--------|----------|
| [项目 1] | [需要人工验证的原因] | High / Medium / Low |
| [项目 2] | [需要人工验证的原因] | High / Medium / Low |

---

## Risks

- ⚠️ [风险 1]: [描述]
- ⚠️ [风险 2]: [描述]

---

## Recommendation

### Status Definition (v3.1 - Powered by mr-review)

| Status | Condition | Action |
|--------|-----------|--------|
| PASS | 无越界、无 Forbidden、`mr-review` 无 Confirmed Bugs、**无未验证关键项** | ✅ 边界与代码质量检查均通过，可进入提交前人工审查 |
| WARNING | 无越界、无 Forbidden，`mr-review` 无 Confirmed Bugs，**但存在未验证项、部分修复或 review skill 缺失/安装失败** | ⚠️ 必须人工验证 Unverified Items 后，再决定是否进入提交前人工审查 |
| FAIL | 触碰 Forbidden、超出 Allowed、违反 Locked Plan、`mr-review` 发现 Confirmed Bugs、或新增未 Allowed 文件 | ❌ 必须停止，并根据风险决定回滚或返回 rr analyze |

### Hard Rule

**如果存在 Unverified Items，状态不能是 PASS，只能是 WARNING 或 FAIL。**

### Final Recommendation

- PASS：边界与代码质量检查均通过，可进入提交前人工审查
- WARNING：必须人工验证 Unverified Items 后，再决定是否进入提交前人工审查
- FAIL：必须停止，并根据风险决定回滚或返回 rr analyze

---

## Generated From

- BOUNDARY.md: [引用版本]
- TASK_PACKET.md: [引用版本]
- Generated: [timestamp]
