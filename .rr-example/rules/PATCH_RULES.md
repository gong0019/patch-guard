# PATCH_RULES.md

长期 Regression Rules，跨 patch 复用。

**规则编号说明**：
- RR-XXX 格式
- 只有重大/典型 regression 才写入
- 不是每个 bug 都沉淀

---

## RR-001

修改 `variants` 相关代码时：

必须同步检查：
- `update_ary` 函数
- `insert_ary` 函数
- `ext_save_data` 数据结构

**原因**：历史 regression：修改 variants 后导致 save pipeline 失效。

---

## RR-002

修改 `tmp_1` 临时变量时：

必须同步检查：
- JS `replace` 链式调用
- `input name` 属性
- `default object` 初始化

**原因**：tmp_1 被多处引用，修改易导致数据流断裂。

---

## RR-003

涉及 `warehouse` 仓库模块时：

禁止：
- 只修改基础区数据

必须同步检查：
- `ProInfo` 产品信息
- `warehouseSet` 仓库设置

**原因**：warehouse 数据结构关联复杂，局部修改易导致数据不一致。

---

## RR-004

修改 `save pipeline` 时：

禁止：
- 修改已有 save 流程顺序
- 删除中间验证步骤

必须：
- 保持 save 流程完整性
- 新增步骤放在流程末端

**原因**：save 流程顺序被多次 regression 验证，不可随意调整。

---

## RR-005

涉及 `workflow` 工作流时：

禁止：
- 直接修改 workflow 核心逻辑

必须：
- 通过配置扩展
- 保持 workflow 状态机完整性

**原因**：workflow 是核心业务逻辑，直接修改风险极高。

---

## 添加新规则

新规则添加条件：
1. 重大 regression，未来可能复现
2. 典型错误模式，其他 AI 可能重犯
3. 核心模块，修改风险高

用户明确要求时才添加。

格式：

```markdown
## RR-XXX

修改 [module/file] 时：

必须同步检查：
- [相关模块 1]
- [相关模块 2]

禁止：
- [禁止行为 1]

必须：
- [必须行为 1]

**原因**：[历史 regression 描述]
```

---

## 规则索引

| RR | Module | Risk Level |
|----|--------|------------|
| RR-001 | variants | High |
| RR-002 | tmp_1 | Medium |
| RR-003 | warehouse | High |
| RR-004 | save pipeline | High |
| RR-005 | workflow | Critical |