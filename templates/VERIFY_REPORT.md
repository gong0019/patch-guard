# RR Verify Report

## Status

[PASS / WARNING / FAIL]

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

### Status Definition (v1.1)

| Status | Condition | Action |
|--------|-----------|--------|
| PASS | 无越界、无 Forbidden、**无未验证关键项** | ✅ 可提交 |
| WARNING | 无越界、无 Forbidden，**但存在未验证项** | ⚠️ **必须人工验证后才能提交** |
| FAIL | 触碰 Forbidden、超出 Allowed、违反 Locked Plan、或新增未 Allowed 文件 | ❌ 必须回滚，返回 Phase 1 |

### Hard Rule

**如果存在 Unverified Items，状态不能是 PASS，只能是 WARNING 或 FAIL。**

### Final Recommendation

- [PASS]: 边界检查通过，可提交
- [WARNING]: 边界检查通过，但存在未验证项，**必须人工验证后才能提交**
- [FAIL]: 边界检查失败，必须回滚并返回 Phase 1

---

## Generated From

- BOUNDARY.md: [引用版本]
- TASK_PACKET.md: [引用版本]
- Generated: [timestamp]