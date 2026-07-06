---
name: ds:build
description: 版本交付 — 注入角色上下文到 openspec，Explore+Spec → Demo → Build 四步流程
---

# /ds:build — 版本交付

## 入口

用户输入 `/ds:build` 触发。

## 前置准备

1. 检查 `solution/vision.md` 是否存在。不存在 → 提示用户先执行 `/ds:vision`
2. 从 `plugin-state.json` 读取 `currentVersion`，自动推断下一版本：
   - 首次 /ds:build → v1.0
   - 默认递增末位 → v1.0 → v1.1
   - 大版本升级（如 2.0）在 Explore 阶段由用户指定
3. 创建版本目录 `solution/v{x}/`

## 流程

### Step 1: Explore + Spec（openspec，一次连贯完成）

**目的：** 利用 openspec 的 explore→propose 链路，注入多角色视角，产出 specification.md（= PRD）。

**操作：**

1. 组装角色要求上下文：

```markdown
## 本次版本上下文（由 AI Team 插件注入）

**版本号：** v{x}
**战略背景：** [从 solution/vision.md 提取关键信息]

### 角色视角要求

**CEO 视角（商业）：**
- 本次版本与产品战略是否对齐？
- 版本目标的商业优先级（MoSCoW）
- 是否值得现在做？

**PO 视角（产品）：**
- 需求范围是否清晰？
- 用户价值是否明确？
- 版本目标是否可度量？

**TL 视角（技术）：**
- 技术可行性如何？
- 是否存在关键技术风险？
- 是否有架构约束需要考虑？

**QA 视角（质量）：**
- 验收标准是否可验证？
- 边界条件是否明确？
```

2. 调用 openspec：`/opsx:explore` → `/opsx:propose`
   - 上述角色上下文作为 openspec explore 的输入
   - openspec propose 产出的 specification.md = PRD
   - 探索和提案在 openspec 内部连贯执行，中间不暂停

3. openspec propose 完成后，将 specification.md 写入 `solution/v{x}/specification.md`

4. **用户确认 spec** — 展示核心内容摘要，等待用户确认

### Step 2: Demo（ui_ux_max_pro）

**目的：** 基于确认的 spec 制作 HTML 高保真原型，让用户确认交互和视觉效果。

**操作：**

1. **加载 PO 角色 Skill（roles/po.md）** 中的原型设计规范
2. **加载 QA 角色 Skill（roles/qa.md）** 中的原型检查标准
3. 读取 `solution/v{x}/specification.md`
4. 使用 `ui_ux_max_pro` 技能输出 HTML 原型
5. 原型输出到 `solution/v{x}/demo/`
6. QA 检查：原型与 spec 一致性、三态覆盖、交互完整性
7. **用户确认原型** — 原型可直接在浏览器打开体验

### Step 3: Build（openspec apply + superpowers）

**目的：** 基于确认的 spec 和原型进行编码交付。

**操作：**

1. **加载 TL 角色 Skill（roles/tl.md）** 中的工程规范
2. **加载 QA 角色 Skill（roles/qa.md）** 中的质量检查标准
3. 编写技术方案 `solution/v{x}/tech-design.md`
4. 调用 openspec：`/opsx:apply`
   - tasks.md 内嵌 TDD 步骤（先写测试 → FAIL → 实现 → PASS）
   - 使用 `superpowers:subagent-driven-development` 驱动开发
   - 每个任务 TDD（`superpowers:test-driven-development`）
   - 每个任务完成后两阶段审查：spec 合规 + 代码质量
   - 所有任务完成后触发 `superpowers:finishing-a-development-branch`
5. QA 贯穿：代码质量、安全、架构合规、测试覆盖率
6. **用户确认交付** — 全量测试通过后才能声称完成

## 新版本 vs 迭代版

| 环节 | 新版本 | 迭代版 |
|------|--------|--------|
| Explore | 战略对齐，定义版本目标 | 范围锁定，影响评估 |
| Spec | 完整 specification.md | 变更 specification.md（diff 视角） |
| Demo | 完整原型 | 变更页面原型 |
| Build | 完整功能开发 | 增量修改 + 回归测试 |
| QA 重点 | 架构合规、测试覆盖 | 回归覆盖、兼容性 |

## 强制规则

- 每步完成后必须等待用户确认才能进入下一步
- Explore+Spec 由 openspec 连贯执行，不在中间插入确认点
- Demo 环节是 spec 确认后、编码前的必要步骤（前端项目）
- 如果项目无前端界面，Demo 步骤可以跳过（由用户在 Explore 时确认）
