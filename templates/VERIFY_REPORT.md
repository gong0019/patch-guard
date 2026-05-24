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
| 添加新功能 | ✅ none / ❌ detected |
| 修改代码风格 | ✅ none / ❌ detected |
| 重命名 | ✅ none / ❌ detected |

**Result**: [✅ none / ❌ detected: ...]

---

## Unverified Items

以下项目尚未验证：

- [ ] [未验证项 1]: [需要验证的内容]
- [ ] [未验证项 2]: [需要验证的内容]
- [ ] [未验证项 3]: [需要验证的内容]

---

## Risks

- ⚠️ [风险 1]: [描述]
- ⚠️ [风险 2]: [描述]

---

## Recommendation

### Status Definition

| Status | Condition | Action |
|--------|-----------|--------|
| PASS | 全部 Allowed，无 Forbidden，风险可控 | ✅ 可提交 |
| WARNING | 全部 Allowed，但有未验证项 | ⚠️ 需补充验证后提交 |
| FAIL | 有超出 Allowed 或触碰 Forbidden | ❌ 必须回滚，返回 Phase 1 |

---

## Generated From

- BOUNDARY.md: [引用版本]
- TASK_PACKET.md: [引用版本]
- Generated: [timestamp]