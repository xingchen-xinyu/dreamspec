# AI Team — AI Agent 创业开发团队

将 openspec、superpowers、ui_ux_max_pro 等技能的开发流程和角色协作方法固化为一个 Claude Code 插件。通过 CEO/PO/TL/QA 四个 AI 角色按标准流程协作，帮你把想法从战略规划到编码交付完整落地。

## 安装

```bash
# Step 1: 注册 marketplace
claude plugin marketplace add xingchen-xinyu/dreamspec --scope project
# Step 2: 刷新 marketplace 缓存（获取最新版本）
claude plugin marketplace update dreamspec-market
# Step 3: 安装插件
claude plugin install dreamspec@dreamspec-market --scope project
```

安装后自动注册以下命令：`/ds:init`  `/ds:upgrade`  `/ds:vision`  `/ds:build`

依赖插件 openspec、superpowers、ui_ux_max_pro、frontend-design 由 `/ds:init` 或 `/ds:upgrade` 自动检测并安装——两个命令都能处理依赖，先跑哪个都行。

## 升级

`/ds:upgrade` 是 DreamSpec 的依赖管家，支持两种升级模式。`/ds:init` 也具备依赖安装能力——两个命令都能处理依赖，先跑哪个都行。

### 使用 /ds:upgrade（推荐）

在 Claude Code 中运行：

```
/ds:upgrade       # 启动后上下箭头选择升级模式，回车确认
```

启动后通过选项菜单选择模式（上下箭头切换，回车确认）：

| 模式 | 说明 | 适用场景 |
|------|------|---------|
| **A. 全量更新** | 检测所有依赖 + 升级全部插件 | 首次使用、久未升级、不确定依赖状态 |
| **B. 仅升级 DreamSpec** | 跳过依赖检测，只升级主插件 | 日常跟进、依赖已就绪、追求速度 |

### 全量更新（模式 A）

1. 检测并自动安装缺失的依赖插件（superpowers、frontend-design、ui_ux_max_pro、openspec）
2. 检测 dreamspec 主插件是否有新版本（从 GitHub 获取远端版本号）
3. 检测所有依赖插件是否有更新
4. 展示版本差异汇总表，等待你确认
5. 依次升级依赖插件，最后升级 dreamspec 自身
6. （已初始化项目）升级后合规复检，提示是否需要同步项目结构

### 仅升级 DreamSpec（模式 B）

1. 获取 dreamspec 远端最新版本号
2. 展示版本对比，确认后直接升级
3. （已初始化项目）合规复检

模式 B 跳过依赖检测，执行更快，适合日常跟进。

### 强制升级

如果升级过程中出现问题，或需要强制覆盖本地文件：

```
/ds:upgrade --force      # 全量更新模式下强制升级，用远端最新版本覆盖本地 Skill 文件，保留用户数据
```

### 手动升级（备选）

```bash
# 更新 marketplace 和插件到最新版本
claude plugin marketplace update dreamspec-market
claude plugin update dreamspec@dreamspec-market --scope project
```

然后在项目目录中运行 `/ds:init` 同步目录结构和配置（或运行 `/ds:upgrade` 自动完成合规复检）。

## 卸载

```bash
claude plugin uninstall dreamspec@dreamspec-market
```

## 快速开始

### 全新项目

```bash
mkdir my-app && cd my-app
claude plugin marketplace add xingchen-xinyu/dreamspec --scope project
claude plugin marketplace update dreamspec-market
claude plugin install dreamspec@dreamspec-market --scope project
```

在 Claude Code 中（先跑 `/ds:init` 或 `/ds:upgrade` 都行，两个命令都会自动安装缺失依赖）：

```
/ds:init          # 配置项目信息，创建目录结构，合规检测
/ds:vision        # 精简访谈 + 分步确认，制定产品定位与战略
接下来直接在对话中描述你想做的事，build 会自动触发
```

### 已有项目

```bash
cd existing-project
claude plugin marketplace add xingchen-xinyu/dreamspec --scope project
claude plugin marketplace update dreamspec-market
claude plugin install dreamspec@dreamspec-market --scope project
```

在 Claude Code 中（先跑 `/ds:init` 或 `/ds:upgrade` 都行）：

```
/ds:init          # 检测项目状态，无痛迁移，不删除任何现有文件
接下来直接在对话中描述你想做的事，build 会自动触发
```

## 命令参考

| 命令 | 用途 | 何时用 |
|------|------|--------|
| `/ds:init` | 项目初始化/迁移 + 合规检测 | 新项目开始、切换到插件模式（自动安装缺失依赖）|
| `/ds:upgrade` | 依赖管理 + 插件升级 | 支持全量更新（含依赖）和仅升级 DreamSpec 两种模式，支持 `--force` 强制覆盖 |
| `/ds:vision` | 产品定位梳理 | 0→1 必须：精简访谈 → 分步确认，产出产品宪法（含待细化方向），后续按需调整战略 |
| `/ds:build` | 版本交付 | 自动触发，统一承载功能开发/优化/重构/修复，开发→发版两阶段 |

## 工作流

### /ds:build 两种模式

**开发期**：用户表达开发意图 → 自动触发 build → 意图路由 → 工作项累积到 unreleased
**发版期**：用户说"发版" → 汇总所有工作项 → 自动判定版本类型 → 生成版本档案

```
自动触发 build
    ├── 大版本 → CEO 审核 → 完整四步（Explore+Spec → Demo → Build）→ 集成稳定 → 发版
    ├── 功能/优化/重构 → Explore+Spec → Demo(可选) → Build → 工作项完成
    ├── 修复 → TL定位 → 修复 → QA验证 → 工作项完成
    └── 发版 → 汇总 unreleased → 判定 major/minor/patch → 生成档案 + git tag
```

每步完成后用户确认，QA 贯穿全流程。

## 角色说明

| 角色 | 负责 | 参与场景 |
|------|------|---------|
| **CEO** | 商业价值、战略方向 | `/ds:vision`（主导）、`/ds:build` Explore + Major 审核 |
| **PO** | 产品设计、Spec、原型 | `/ds:build` Spec + Demo（主导） |
| **TL** | 技术方案、编码交付、问题修复 | `/ds:build` Build + 修复流程（主导） |
| **QA** | 质量审查、规范检查 | 全流程贯穿 |

## 目录结构

```
项目根目录/
├── solution/                   # 战略、版本档案
│   ├── vision.md               # 产品宪法
│   ├── unreleased/             # 开发期工作项
│   └── v{major}/               # 已发版（按 Major 分组）
│       └── v{x.y.z}/
│           ├── specification.md  # 版本规格（产品决策视角）
│           ├── tech-design.md    # 技术方案
│           ├── demo/             # HTML 原型
│           ├── delivery-report.md # 交付报告
│           └── changelog.md      # 发布说明
├── src/
│   ├── server/        # 后端
│   ├── web/           # 前端
│   └── ...            # /ds:vision 后按技术栈扩展
├── .claude/
│   └── plugin-state.json
└── README.md
```

## FAQ

**Q: 和直接使用 openspec/superpowers 有什么区别？**

A: 本插件是它们的上层编排——提供角色视角、阶段划分、协作流程。你不再需要自己记住"什么时候用哪个技能"。

**Q: 已有项目会不会被破坏？**

A: `/ds:init` 和 `/ds:upgrade` 只新增文件和目录，不删除、不修改任何现有文件。原 CLAUDE.md 会备份为 `.bak`。两个命令都具备依赖安装能力，无论先跑哪个都能确保环境就绪。

**Q: 每个阶段必须走完吗？**

A: 每步完成后你可以选择跳过或调整。流程是骨架，你是决策者。

**Q: 无前端的后端项目 Demo 步骤怎么处理？**

A: build 的 Explore 阶段会自动判定——feature 询问用户是否需要 Demo，optimization/refactor/fix 默认跳过。纯后端项目可省略。
