# CORE_MODULES.md

## Purpose

列出项目的核心模块，AI 在修改这些模块时需要特别注意。

---

## Core Modules List

| Module | Reason | Severity |
|--------|--------|----------|
| ProductEditor.vue | 产品编辑核心组件 | High |
| ext_save_data.js | 保存数据流核心 | High |
| warehouse/ | 仓库模块，数据关联复杂 | High |

---

## Rules

1. **AI must not assume a module is core**
2. A module is core only if listed in this file or confirmed by human during Analyze
3. If this file does not exist, AI may suggest a core module candidate, but cannot use "core module" as promotion evidence without human confirmation

---

## Notes

- 此文件由用户维护
- AI 只能建议候选，不能自行添加
- 定期 review 有效性