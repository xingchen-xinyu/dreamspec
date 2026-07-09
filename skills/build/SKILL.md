---
name: ds:build
description: 版本交付 — 自动触发，意图路由，统一承载功能/优化/重构/修复，两阶段（开发期→发版期）模型
---

# /ds:build — 版本交付

## 触发机制

### 自动触发（主要方式）

用户在项目目录下表达以下意图时，AI **自动使用 Skill 工具加载本技能**，用户无需显式输入 `/ds:build`：

| 意图类型 | 用户表述示例 |
|---------|------------|
| 功能开发 | "加一个导出功能"、"做一个用户管理页"、"实现登录" |
| 问题修复 | "这个按钮点不了"、"登录页布局乱了"、"报 500 错误帮我看看" |
| 性能优化 | "列表加载太慢了"、"首页首屏性能优化一下" |
| 代码重构 | "把用户模块重构一下"、"这坨代码太乱了整理一下" |
| 继续开发 | "继续做权限功能"、"接着昨天的做" |
| 发版 | "发版"、"发布版本"、"可以上线了"、"打包发布" |

### 不触发的情况（红线）

以下情况**禁止自动加载** build 技能，直接正常响应：

- 纯问答/了解代码："这个函数干什么的"、"代码结构是怎样的"
- 查看/探索："帮我看看这个文件"、"查一下哪里用了这个接口"
- 闲聊/讨论："你觉得这个方案怎么样"、"我想讨论一下架构"
- **用户显式调用其他命令**：`/ds:init`、`/ds:upgrade`、`/ds:vision`、以及任何第三方 skill
- Git 操作："帮我提交代码"、"切个分支"
- 用户明确跳过："不用走 build 流程"、"直接改就行"

### 显式调用（兼容）

用户输入 `/ds:build [描述]` 触发，与自动触发行为一致。

---

## 入口路由

build 技能被调起后，按以下决策树路由（按顺序匹配，命中即停止）：

```
build 技能入口
    │
    ├── 用户意图 = "发版" / "发布" / "打包"（且非"我要开发 X.0"等版本目标意图）
    │   └── 走「流程 E：发版」
    │
    ├── 用户意图 = "开始做" / "开发" + 版本号（如"开发 1.0"/"开始 v2.0"）
    │   └── 走「流程 A：版本目标声明」
    │
    ├── 用户意图 = "继续" / "接着做" / "接着昨天的" / "做完" / "完成"
    │   └── 走「流程 D：继续已有工作项」
    │
    ├── 用户意图 = "取消" / "废弃" / "不做了" + 指定工作项
    │   └── 标记工作项为 cancelled，从 unreleased 移除
    │
    ├── 用户描述 = 微小调整（"改一下" / "调一下" / "换一下"，改动明确、不涉及逻辑）
    │   └── 走「微小调整」：直接修改 → 口头确认效果（不建工作项，不走诊断和 QA）
    │
    ├── 用户描述 = 功能开发 / 优化 / 重构
    │   └── 走「流程 B：完整流程」
    │
    ├── 用户描述 = 修复 / bug / 报错 / 有问题
    │   └── 走「流程 C：Fix 简化流程」
    │
    └── 无描述 / 意图不明确
        ├── 有 in_progress 工作项 → "检测到进行中的「xxx」，继续吗？"
        └── 无进行中工作项 → "想做点什么？新功能 / 修问题 / 优化改进？"
```

---

## 前置准备（所有流程共用）

1. 检查 `solution/vision.md` 是否存在。不存在 → 提示用户先执行 `/ds:vision`
2. 读取 `.claude/plugin-state.json`，获取：
   - `currentVersion`：当前生产版本号
   - `versionGoal`：当前版本战略目标（如 "1.0.0"），为 Explore 注入 vision 上下文
   - `versionPlan`：版本演进计划文件路径
   - `worklog.unreleased`：未发版工作项列表
3. 如果 `versionGoal` 非 null → Explore 阶段自动注入 vision 中对应的战略上下文
4. 意图识别 → 路由到对应流程

---

## 流程 A：版本目标声明

### 触发

用户表达明确的版本开发意图："我要开始做 1.0"、"开始开发 v2.0"、"v2.0 版本"。

> 注意区分："我要开发 2.0"（声明版本目标，走流程 A）vs "发版"（发布当前累积，走流程 E）。

### Step A1: CEO 战略对齐

1. **加载 CEO 角色 Skill（roles/ceo.md）**
2. 读取 `solution/vision.md`，找到对应版本的路线图规划
3. CEO 输出战略上下文摘要：
   - 这个版本在 vision 路线图中的定位和北极星指标
   - 如果是计划外的版本目标，什么触发了它？
   - 战略对齐度评估 + 关键风险提示
4. **用户确认** 版本目标方向

### Step A2: 探索 MVP + 演进计划

**目的：** 与用户讨论确定 MVP 范围 + 版本演进路线。

1. 调用 openspec 基础流程：`/opsx:explore`
   - explore 聚焦三个核心问题：
     - **MVP 边界**：最先交付的最小可用集是什么？什么放到后面？
     - **版本拆分**：预计分几个小版本？每个版本大概安排哪些内容？
     - **优先级**：功能和需求之间的依赖关系和先后顺序
2. 与用户讨论并确认：

   > 基于 explore 的结果，我建议这样规划 v1.0：
   >
   > **MVP（v0.1）**：
   > - [最核心的功能 1]
   > - [最核心的功能 2]
   > - Out of MVP：[暂时不做的]
   >
   > **后续迭代**：
   > - v0.2：[功能 A、B]
   > - v0.3：[功能 C、D]
   > - v1.0：全功能集成 + 性能稳定
   >
   > 这只是初始计划，每个小版本做完后可以根据实际情况调整。你觉得这个拆分合理吗？

3. 用户确认后，产出：
   - 工作项级 `spec.md`（MVP 首个迭代的 spec）→ `solution/unreleased/{id}/spec.md`
   - 版本演进计划 `solution/version-plan.md`

### Step A3: 写入版本演进计划

生成 `solution/version-plan.md`（活文档，会随实际推进更新）：

```markdown
# v{major} 版本演进计划

> versionGoal: {版本号} | 创建: {日期} | 更新: {日期} | 状态: 进行中

## 战略目标
[从 vision.md 提取对应的战略上下文]

## MVP 范围
[MVP 要做和暂时不做的内容]

## 演进计划 vs 实际

| 版本 | 计划内容 | 实际交付 | 状态 |
|------|---------|---------|------|
| v0.1.0 | [计划] | — | 📋 |
| v0.2.0 | [计划] | — | 📋 |
| ... | ... | — | 📋 |
| v{x}.0.0 | 全功能集成、稳定发版 | — | 🎯 |

> 计划列 = 初始规划；实际列 = 每次发版后回填真实内容。
> 计划变更时直接更新"计划内容"列，保留变更痕迹。
```

### Step A4: 更新 plugin-state.json

```json
{
  "versionGoal": "1.0.0",
  "versionPlan": "solution/version-plan.md"
}
```

汇报：

> v1.0 版本目标已设定。MVP 从 v0.1 开始，后续逐步迭代。
> 从现在开始，所有开发都会自动带入 v1.0 的战略上下文。
> 演进计划保存在 solution/version-plan.md，每次发版后自动更新。
> 要现在开始 MVP 的第一个功能吗？比如「{从 MVP 计划中取第一个功能描述}」？

### 后续：正常开发迭代

versionGoal 设定后，进入正常的开发期——每个小版本走「流程 B：完整流程」：

```
v0.1: Explore+Spec（自动注入 v1.0 上下文）→ Demo → Build → 发版
v0.2: Explore+Spec（自动注入 v1.0 上下文）→ Demo → Build → 发版
v0.3: ...
v1.0: CEO 战略检查 → 发版 → versionGoal 清空
```

每个小版本跟日常开发完全一样，区别仅在于：
- Explore 阶段自动注入 `versionGoal` 对应的 vision 战略上下文
- 发版后自动更新 `version-plan.md` 的"实际交付"列

### 版本目标期间

- **不阻塞发版**：随时可以发版，不锁定在某个版本号
- **发版自动回填**：每次发版后更新 version-plan.md，"实际交付"列记录真实内容
- **计划可变**：用户在后续迭代中可以调整演进计划，直接更新 version-plan.md 的"计划内容"列
- **达到目标时**：用户说"发 1.0" → CEO 对照 vision 检查目标达成 → 发版 → `versionGoal` 清空

---

## 流程 B：完整流程（Feature / Optimization / Refactor）

适用于功能开发、性能优化、代码重构。走 Explore+Spec → Demo(可选) → Build 三步。

### Step B1: 创建工作项

在 `plugin-state.json` 的 `worklog.unreleased` 中创建：

```json
{
  "id": "wl-{YYYYMMDD}-{序号}",
  "scope": "{versionGoal 或 null}",
  "type": "feature|optimization|refactor",
  "title": "{用户描述摘要}",
  "summary": "",
  "createdAt": "{ISO timestamp}",
  "completedAt": null,
  "status": "in_progress",
  "sessions": ["{当前 session ID}"],
  "commits": [],
  "files": [],
  "workDir": "solution/unreleased/{id}/"
}
```

创建目录 `solution/unreleased/{id}/`。

### Step B2: Explore + Spec（openspec，一次连贯完成）

**目的：** 利用 openspec 的 explore→propose 链路，注入多角色视角，产出 spec.md。

1. 组装角色要求上下文：

```markdown
## 本次工作上下文（由 DreamSpec 注入）

**工作项 ID：** {wl-xxx}
**类型：** {feature/optimization/refactor}
**版本归属：** {versionGoal 或 "待分配"}
**战略背景：** [从 solution/vision.md 提取关键信息]

### 角色视角要求

**CEO 视角（商业）：**
- 本次工作与产品战略是否对齐？
- 优先级评估（MoSCoW）

**PO 视角（产品）：**
- 需求范围是否清晰？
- 用户价值是否明确？
- 目标是否可度量？

**TL 视角（技术）：**
- 技术可行性如何？
- 是否存在关键技术风险？
- 是否有架构约束需要考虑？

**QA 视角（质量）：**
- 验收标准是否可验证？
- 边界条件是否明确？
```

2. 调用 openspec 基础流程：`/opsx:explore` → `/opsx:propose`
   - 上述角色上下文作为 openspec explore 的输入
   - 探索和提案在 openspec 内部连贯执行，中间不暂停
   - 所有场景（feature/optimization/refactor）统一走此基础流程，不区分 new/continue/ff

3. 基于 openspec 输出，生成工作项级别的 `spec.md`，写入 `solution/unreleased/{id}/spec.md`：

```markdown
# {工作项标题}

- **工作项 ID：** {wl-xxx}
- **类型：** {feature/optimization/refactor}
- **版本归属：** {scope 或 "待分配"}
- **创建日期：** {date}

## 需求概述
[用户价值 + 核心功能点]

## 范围
| 优先级 | 内容 |
|--------|------|
| Must | ... |
| Should | ... |
| Out of Scope | ... |

## 验收标准
[可验证的验收条件]

## 技术要点
[关键技术决策和风险]

## 关联 OpenSpec
[proposal 路径]
```

4. **用户确认 spec** — 展示核心内容摘要，等待用户确认

### Step B3: Demo 判定

根据工作项类型判定是否需要 Demo：

| 类型 | Demo 策略 |
|------|----------|
| feature | **询问用户**是否需要原型（默认建议需要，除非纯后端/无 UI） |
| optimization | **默认跳过**，除非用户明确要求 |
| refactor | **默认跳过**（重构不改 UI），除非用户明确要求 |

如果用户确认需要 Demo，或项目有前端界面且 feature 类型：

1. **加载 PO 角色 Skill（roles/po.md）** 中的原型设计规范
2. **加载 QA 角色 Skill（roles/qa.md）** 中的原型检查标准
3. 使用 `ui_ux_max_pro` 技能输出 HTML 原型到 `solution/unreleased/{id}/demo/`
4. QA 检查：原型与 spec 一致性、三态覆盖、交互完整性
5. **用户确认原型**

如果跳过 Demo，记录跳过原因到 spec.md。

### Step B4: Build（openspec apply）

1. **加载 TL 角色 Skill（roles/tl.md）** 中的工程规范
2. **加载 QA 角色 Skill（roles/qa.md）** 中的质量检查标准
3. 编写技术方案记录到 spec.md 的技术要点章节
4. 调用 openspec：`/opsx:apply`
   - tasks.md 内嵌 TDD 步骤（先写测试 → FAIL → 实现 → PASS）
   - 使用 `superpowers:subagent-driven-development` 驱动开发
   - 每个任务 TDD（`superpowers:test-driven-development`）
   - 每个任务完成后两阶段审查：spec 合规 + 代码质量
   - 所有任务完成后触发 `superpowers:finishing-a-development-branch`
5. QA 贯穿：代码质量、安全、架构合规、测试覆盖率

### Step B5: 收尾

1. **OpenSpec 收尾（连贯执行）：**
   - `/opsx:verify` — 验证实现与 spec 的一致性
     - 检查所有 tasks 是否完成
     - 检查验收标准是否全部满足
     - 如有不一致 → 列出差异项 → 修复后重试
   - `/opsx:sync` — 将代码变更同步回 spec，消除 drift
   - `/opsx:archive` — 归档已完成的 proposal

2. 更新 `plugin-state.json`：
   - `status` → `completed`
   - `completedAt` → 当前时间
   - `summary` → 实际交付内容摘要
   - `commits` → 关联的 git commit 列表
   - `files` → 修改的文件/目录列表

3. 汇报完成

---

## 流程 C：Fix 简化流程

适用于 bug 修复、线上问题处理。走 TL定位 → 修复 → QA验证 三步，不产生 spec 和 Demo。

### Step C1: 创建工作项

在 `plugin-state.json` 的 `worklog.unreleased` 中创建，`type: "fix"`。

创建目录 `solution/unreleased/{id}/`。

### Step C2: TL 定位

**加载 TL 角色 Skill（roles/tl.md）。**

1. 使用 `superpowers:systematic-debugging` 技能：
   - 复现问题
   - 诊断根因（不满足于表面现象）
   - 分析影响范围
2. 口头汇报诊断结果，包含：
   - 问题描述和复现步骤
   - 根因分析（代码层面的根本原因）
   - 影响范围（哪些功能/用户受影响）
   - 修复方案（具体做法 + 优缺点）
3. **用户确认修复方案**

### Step C3: TL 修复

1. 先写复现测试 → 确认 FAIL（证明 bug 存在）
2. 修复代码 → 测试 PASS
3. 运行回归测试 → 确保无副作用
4. 使用 `superpowers:test-driven-development` 技能保障 TDD 流程

### Step C4: QA 验证

**加载 QA 角色 Skill（roles/qa.md）。**

1. 检查修复完整性：
   - 根因是否被真正修复（不是绕过）
   - 边界条件是否覆盖
2. 检查无副作用：
   - 回归测试全部通过
   - 相关功能正常
3. 口头汇报 QA 验证结论
4. 使用 `superpowers:verification-before-completion` 技能验证

**循环规则：** 如果 QA 发现阻塞问题，回到 Step C2 TL 定位，最多循环 3 次。3 次后仍未通过，标记问题升级。

### Step C5: 完成工作项

> Fix 流程无 openspec proposal（跳过 explore→propose），因此无需 verify/sync/archive。

1. 更新 `plugin-state.json`：
   - `status` → `completed`
   - `completedAt` → 当前时间
   - `summary` → 修复内容摘要
   - `commits` → 关联的 git commit 列表
   - `files` → 修改的文件/目录列表
2. 汇报完成

---

## 微小调整

适用于用户随口提出的琐碎改动，语气随意、范围明确、不涉及业务逻辑。

**典型表述：** "按钮颜色改一下"、"这个文字换成 XX"、"间距调大一点"

**流程（极简）：**
1. 直接修改代码
2. 展示改动效果 → 口头确认即可

**不做的事：** 不创建工作项、不走 TL 诊断、不走 QA 验证、不生成任何文档。

> 如果用户说"直接改就行"、"不用走 build 流程" → build 技能本身不会被触发，效果相同。

---

## 流程 D：继续已有工作项

### Step D1: 匹配工作项

1. 扫描 `worklog.unreleased`，查找匹配：
   - 用户说"继续XXX"/"做完XXX"/"完成XXX" → 关键词匹配 `title` 字段
   - 修改文件与已有工作项 >70% 重叠 → 提示确认是否为同一项
   - 标题相似度 >70% → 提示确认

2. 匹配成功 → 展示匹配结果：
   > 检测到相关工作项「{title}」(wl-xxx)，状态：{status}。要继续做这个吗？

3. 用户确认后：
   - `status` → `in_progress`（如果已完成则重新打开）
   - `sessions` 追加当前 session
   - 走 `type` 对应的流程（B 或 C）

4. **无匹配 → 兜底**：
   > 没有找到相关的进行中工作项。作为新需求创建？
   - 用户确认 → 走入口路由重新判定（流程 B 或 C）
   - 用户拒绝 → 结束

### 去重规则

| 场景 | 判定 | 行为 |
|------|------|------|
| 用户说"继续XXX" | 关键词匹配 | 追加到已有工作项 |
| 修改文件 >70% 重叠 | 文件路径交集 | 提示确认后追加 |
| 标题相似 >70% | 文本匹配 | 提示确认后追加 |
| 都不匹配 | 新工作项 | 创建 wl-xxx，走流程 B/C |

---

## 流程 E：发版

用户表达发版意图时触发。

### Step E1: 汇总待发版内容

1. 扫描 `worklog.unreleased` 中所有 `status: "completed"` 的工作项
2. 如果有 `status: "in_progress"` 的工作项，提示：
   > ⚠️ 检测到进行中的工作项「{title}」。发版不会包含它。确认继续发版？
3. 如果没有 completed 工作项 → "没有待发版的内容。"
4. 展示汇总：

```
📦 待发版内容（{N} 项）：

• [feature] 用户权限管理 — 2026-07-09
• [fix] 移动端登录布局修复 — 2026-07-08
• [optimization] 用户列表查询性能优化 — 2026-07-07

当前版本：v{currentVersion}
```

### Step E2: 判定版本类型

```
if currentVersion == "0.0.0"（首次发版）:
    → 询问用户："正式发布 v1.0.0 还是预览版 v0.1.0？"

elif 用户明确指定版本号（如"发 1.0"/"这次叫 v2.0"）:
    → 以用户指定为准

elif versionGoal 非 null 且用户说"发 {versionGoal 对应 major}"（如"发 1.0"）:
    → CEO 战略检查（对照 vision 确认目标是否达成）
    → versionGoal 目标版本

elif currentVersion 是 0.x.x 系列:
    → unreleased 全 fix → v0.{y}.{z+1}
    → 含 feature/opt/refactor → v0.{y+1}.0

else（1.0+ 系列，日常开发）:
    → unreleased 全 fix → Patch (v{x}.{y}.{z+1})
    → 含 feature/opt/refactor → Minor (v{x}.{y+1}.0)
```

展示建议版本号，用户可手动调整：

> 建议新版本：**v1.3.0** (Minor)
> 确认？或输入其他版本号：

### Step E3: 版本级汇总检查

1. 检查所有待发版工作项：
   - 确认所有工作项 `status: "completed"`
   - 列出各工作项的 spec.md 和关联的 OpenSpec proposal（如有）
2. 跨工作项一致性检查：
   - 是否有文件/模块被多个工作项修改（潜在的 merge 冲突）
   - 检查各工作项之间是否有未解决的依赖
3. 如有问题 → 列出清单 → 用户决定处理方式（修复 / 记录到 delivery-report / 本次不发）

### Step E4: 生成版本档案

1. 创建版本目录 `solution/v{major}/v{major}.{minor}.{patch}/`
2. 生成 `specification.md`（DreamSpec 版本规格，产品决策视角）：

```markdown
# v{x.y.z} — {版本标题}

## 版本概述
- 版本号：v{x.y.z}
- 版本类型：{major/minor/patch}
- 发布日期：{date}
- 一句话目标：{summary}

## 战略对齐
> 对应 vision.md 路线图：{阶段}
> 北极星指标贡献：{描述}

## 需求范围
| 优先级 | 内容 | 来源工作项 |
|--------|------|-----------|
| Must | ... | wl-xxx |
| Should | ... | wl-xxx |
| Out | ... | — |

### 附带修复
[本次版本包含的 fix 工作项列表]

## 决策记录
[本版本中的关键决策及理由]

## 原型确认
[Demo 版本、确认日期、关键反馈（如适用）]

## 关联
- OpenSpec proposal：[路径]
- Git tag：v{x.y.z}
- 包含工作项：[wl-xxx, wl-yyy, ...]
```

3. 生成 `changelog.md`（面向团队的发布说明）
4. 生成 `delivery-report.md`（交付报告：实际 vs 计划、测试覆盖、已知问题）
5. 移动 unreleased 工作项目录到版本目录下
6. 复制 Demo 文件（如有）到版本目录
7. **如果存在 `versionPlan`**：更新 `solution/version-plan.md`，将本次版本对应行的"实际交付"列回填真实内容，状态更新为 ✅

### Step E5: 收尾

1. 更新 `plugin-state.json`：
   - `currentVersion` → 新版本号
   - `versionGoal` → null（如果发版的版本号与 versionGoal 匹配，说明目标达成）
   - `versions[]` 追加新版本记录
   - `worklog.released[新版本号]` → 本次发版的工作项 ID 列表
   - `worklog.unreleased` → 清空已发版的工作项

2. 打 git tag：`v{x.y.z}`

3. 汇报：
   > ✅ v{x.y.z} 已发布
   > - 包含 {N} 个工作项
   > - 版本档案：solution/v{major}/v{x.y.z}/
   > - Git tag：v{x.y.z}

---

## 工作项模型

### 字段说明

| 字段 | 类型 | 说明 |
|------|------|------|
| `id` | string | 唯一标识，格式 `wl-YYYYMMDD-NNN` |
| `scope` | string\|null | 归属版本目标。null=日常开发无目标；"1.0.0"=标记为 versionGoal 下的工作项 |
| `type` | enum | `feature` / `fix` / `optimization` / `refactor` |
| `title` | string | 简短标题 |
| `summary` | string | 实际交付内容摘要（完成时填写） |
| `createdAt` | ISO datetime | 创建时间 |
| `completedAt` | ISO datetime\|null | 完成时间 |
| `status` | enum | `in_progress` / `completed` / `cancelled` |
| `sessions` | string[] | 关联的会话 ID 列表 |
| `commits` | string[] | 关联的 git commit SHA 列表 |
| `files` | string[] | 修改的文件/目录列表 |
| `workDir` | string | 工作项目录路径 |

### 生命周期

```
创建 (in_progress) → 开发中 → 完成 (completed) → 发版 → 移入 released
                       ↓                    ↕
                    可暂停（跨会话）      可重新打开（继续开发）
                       ↓
                    废弃 (cancelled) → 从 unreleased 移除
```

---

## 目录结构

```
solution/
├── vision.md                        # 产品宪法
├── interview-raw.md                 # 访谈原始记录
├── version-plan.md                  # 版本演进计划（活文档，计划 vs 实际）
├── unreleased/                      # 开发期：未发版工作项
│   ├── wl-20260709-001/             # feature 类
│   │   ├── spec.md                  # 工作项规格
│   │   └── demo/                    # 原型（如适用）
│   └── wl-20260708-001/             # fix 类（无文件，诊断和验证口头汇报）
├── v1/                              # Major 1 系列
│   ├── v1.0.0/                      # 首个 major（完整档案）
│   │   ├── specification.md         # DreamSpec 版本规格
│   │   ├── tech-design.md           # 技术方案
│   │   ├── demo/                    # 原型
│   │   ├── delivery-report.md       # 交付报告
│   │   └── changelog.md             # 发布说明
│   ├── v1.1.0/                      # 功能版本
│   └── v1.2.0/                      # 功能版本
└── v2/                              # Major 2 系列
    └── v2.0.0/
```

**层级规则：**
- `solution/` 下按 Major 版本分组为 `v{major}/`，避免平铺过多
- 每个 Minor/Major 版本独立目录 `v{major}.{minor}.{patch}/`
- **Patch 版本不单独建目录**：patch 更新到对应 minor 目录的 changelog 和 delivery-report
- 发版后 unreleased 工作项目录移到对应版本目录下

---

## specification.md 与 OpenSpec 的关系

| | DreamSpec specification.md | OpenSpec spec |
|---|---|---|
| **视角** | 产品决策："建什么 + 为什么" | 技术执行："怎么建" |
| **读者** | 人（团队复盘、版本追溯） | AI（执行开发任务） |
| **内容** | 版本目标、战略对齐、多角色评审、决策记录 | 详细任务拆解、验收用例、技术细节 |
| **位置** | `solution/v{major}/v{x.y.z}/specification.md` | `openspec/` 目录 |
| **生命周期** | 永久档案，每个版本一份 | 当前版本开发期间有效 |
| **关联** | 通过"关联"章节引用 OpenSpec proposal | 通过 proposal 名称关联版本 |

两者互补，不重复。OpenSpec spec 是"消耗品"（版本交付后使命结束），DreamSpec spec 是"档案"（为什么做、做了什么、谁做的决策）。

---

## plugin-state.json 完整结构

```json
{
  "project": { "name": "", "description": "" },
  "plugin_version": "",
  "currentVersion": "0.3.0",
  "versionGoal": "1.0.0",
  "versionPlan": "solution/version-plan.md",
  "status": "init_completed",
  "phases": {
    "init_completed": true,
    "vision_completed": true
  },
  "techStack": {},
  "directories": {},
  "installed_at": "",
  "versions": [
    {
      "version": "1.0.0",
      "type": "major",
      "date": "2026-06-15",
      "title": "MVP — 核心功能上线",
      "scope": "用户注册、登录、基础数据管理",
      "gitTag": "v1.0.0",
      "specPath": "solution/v1/v1.0.0/specification.md",
      "status": "delivered"
    }
  ],
  "worklog": {
    "unreleased": [],
    "released": {
      "1.0.0": ["wl-20260610-001"]
    }
  }
}
```

---

## 强制规则

- **自动触发边界**：纯问答、代码查看、讨论、显式调用其他 skill → 不触发 build
- **OpenSpec 基础流程**：所有场景统一使用 openspec 基础流程（explore → propose → apply），不引入扩展命令（new/continue/ff）。fix 场景跳过 explore→propose 直接 apply
- **Explore+Spec 连贯执行**：openspec 内部 explore→propose 不插入确认点，完成后统一确认
- **OpenSpec 收尾连贯执行**：每个工作项 apply 完成后，立即执行 verify → sync → archive，proposal 生命周期完整闭环。发版时不再重复 archive
- **Demo 按类型判定**：feature 询问用户；optimization/refactor/fix 默认跳过
- **每步确认**：Explore+Spec 完成后、Demo 完成后、Build 完成后，三处必须等待用户确认
- **版本目标不阻塞发版**：versionGoal 提供战略上下文，但不锁定版本号。用户随时可以发版，版本号在发版时判定
- **版本目标发版检查**：用户发版到 versionGoal 目标版本时，CEO 对照 vision 检查目标达成情况
- **去重**：继续/文件重叠/标题相似 → 匹配已有工作项，不重复创建
- **工作项不可跨版本**：一个工作项只属于一个版本。跨版本场景应拆分工作项
- **Patch 不建目录**：patch 版本更新到对应 minor 目录，不单独创建
- **fix 先诊断后修复**：必须先完成根因诊断和用户确认，再动手写修复代码
- **QA 循环上限**：fix 流程 QA 不通过最多循环 3 次，超过标记为需人工介入
