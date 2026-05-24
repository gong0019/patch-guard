# PATCH_RULES.md

长期 Regression Rules，跨 patch 复用。

---

## Core Principle

**AI may propose RR Candidates, but only human-confirmed rules can be promoted to PATCH_RULES.**

**禁止 AI 自行 promote。**

---

## Promoted Rules

### RR-001

修改 `variants` 相关代码时：

必须同步检查：
- `update_ary` 函数
- `insert_ary` 函数
- `ext_save_data` 数据结构

**原因**：历史 regression：修改 variants 后导致 save pipeline 失效。

**Promoted By**: Human on 2024-05-20

---

### RR-002

修改 `tmp_1` 临时变量时：

必须同步检查：
- JS `replace` 链式调用
- `input name` 属性
- `default object` 初始化

**原因**：tmp_1 被多处引用，修改易导致数据流断裂。

**Promoted By**: Human on 2024-05-21

---

### RR-003

涉及 `warehouse` 仓库模块时：

禁止：
- 只修改基础区数据

必须同步检查：
- `ProInfo` 产品信息
- `warehouseSet` 仓库设置

**原因**：warehouse 数据结构关联复杂，局部修改易导致数据不一致。

**Promoted By**: Human on 2024-05-22

---

### RR-004

修改 `save pipeline` 时：

禁止：
- 修改已有 save 流程顺序
- 删除中间验证步骤

必须：
- 保持 save 流程完整性
- 新增步骤放在流程末端

**原因**：save 流程顺序被多次 regression 验证，不可随意调整。

**Promoted By**: Human on 2024-05-22

---

### RR-005

涉及 `workflow` 工作流时：

禁止：
- 直接修改 workflow 核心逻辑

必须：
- 通过配置扩展
- 保持 workflow 状态机完整性

**原因**：workflow 是核心业务逻辑，直接修改风险极高。

**Promoted By**: Human on 2024-05-23

---

## RR Candidates (Not Promoted)

### C-001

**Status**: Rejected

**Proposed Rule**: 修改 ProductEditor.vue 时，必须检查 variants 回显逻辑。

**Reason for Rejection**: 规则过于具体，只针对此 patch，无法指导未来 patches。

**Rejected By**: Human on 2024-05-24

---

### C-002

**Status**: Deferred

**Proposed Rule**: 涉及多产品类型时，必须验证所有类型。

**Reason for Deferral**: 需要更多 patch 经验验证是否为典型模式。

**Deferred By**: Human on 2024-05-24

---

## Rule Index

### Promoted Rules

| RR | Module | Severity | Summary | Promoted By |
|----|--------|----------|---------|-------------|
| RR-001 | variants | High | 检查 update_ary/insert_ary/ext_save_data | Human |
| RR-002 | tmp_1 | Medium | 检查 replace/input name/default object | Human |
| RR-003 | warehouse | High | 禁止只修改基础区 | Human |
| RR-004 | save pipeline | High | 禁止修改流程顺序 | Human |
| RR-005 | workflow | Critical | 禁止直接修改核心逻辑 | Human |

### RR Candidates

| Candidate | Status | Proposed Rule | Human Decision |
|-----------|--------|---------------|----------------|
| C-001 | Rejected | 检查 variants 回显逻辑 | Rejected - too specific |
| C-002 | Deferred | 验证所有产品类型 | Deferred - need more evidence |

---

## Notes

- 规则数量保持在精简范围
- 规则必须有实际防止 regression 的价值
- 定期 review 规则有效性
- **AI 不能自行 promote**