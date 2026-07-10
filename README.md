# AI Team — AI Agent 创业开发团队

将工程规范、角色协作方法固化为一个 Claude Code 插件。不侵入你的开发过程——你正常使用 openspec、superpowers、ux_ui 等任意工具，AI 自动识别工作类型并按需加载对应角色的工程要求。CEO/PO/TL 三角色随身带，帮你确保工程底线不丢失。

## 安装

```bash
# Step 1: 注册 marketplace
claude plugin marketplace add xingchen-xinyu/dreamspec --scope project
# Step 2: 刷新 marketplace 缓存（获取最新版本）
claude plugin marketplace update dreamspec-market
# Step 3: 安装插件
claude plugin install dreamspec@dreamspec-market --scope project
```

安装后自动注册以下命令：`/ds:init` `/ds:upgrade` `/ds:vision` `/ds:plan` `/ds:commit` `/ds:release`

角色 `PO` / `TL` 在开发过程中自动触发，无需手动调用。

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
/ds:plan          # 讨论版本规划、MVP 范围、路线图
然后正常开发，AI 自动识别工作类型并加载对应角色工程要求
/ds:commit        # 做完一个功能，提交代码并记录
/ds:release       # 功能都交付了，发版
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
然后正常开发，AI 自动识别工作类型并加载对应角色工程要求
```

## 命令参考

| 命令 | 用途 | 何时用 |
|------|------|--------|
| `/ds:init` | 项目初始化/迁移 + 合规检测 | 新项目开始、切换到插件模式（自动安装缺失依赖）|
| `/ds:upgrade` | 依赖管理 + 插件升级 | 支持全量更新（含依赖）和仅升级 DreamSpec 两种模式，支持 `--force` 强制覆盖 |
| `/ds:vision` | 产品定位梳理 | 0→1 必须：精简访谈 → 分步确认，产出产品宪法（含待细化方向），后续按需调整战略 |
| `/ds:plan` | 版本规划 | 讨论 MVP 范围、版本拆分、优先级、路线图 |
| `/ds:commit` | 提交记录 | 日常提交代码，自动总结+红线检查+记录 worklog |
| `/ds:release` | 发版 | 汇总 unreleased → 生成 changelog → 打 tag → 归档 |

## 工作流

### 核心理念：工程要求随身带

DreamSpec 不接管你的开发过程——你正常使用 openspec、superpowers、ux_ui 等任意工具，AI 根据当前工作类型**自动识别并加载**对应角色的工程要求。

```
用户日常开发（自由使用任意 skill / 工具）
        │
        ▼
意图识别 + 角色自动加载（📋 TL·后端 已激活）
        │
角色要求驻留在上下文 → 后续一切操作自动携带
        │
        ▼
第三方 skill（openspec/superpowers/ux_ui/...）
AI 带着角色要求调用它们，DreamSpec 不拦截
```

### 日常开发节奏

```
开发功能 → /ds:commit（记录提交）
修 bug   → /ds:commit
优化重构 → /ds:commit
...
功能都交付了 → /ds:release（发版）
```

三个命令是存档点，开发过程完全自由。

### 角色自动加载

| 你在做什么 | 自动加载 | 带什么要求 |
|-----------|---------|-----------|
| 写后端代码（API/数据库/服务端）| TL·后端 | TDD + 分层架构 + API 规范 + 安全编码 |
| 写前端代码（页面/组件/小程序）| TL·前端 | 组件复用 + CSS 变量 + 三态 + 检查清单 |
| 讨论方案/写 PRD | PO·PRD | 七要素 + 验收覆盖 + 业务规则可执行 |
| 设计原型 | PO·原型 | HTML 高保真 + 三态 + 按钮防重 |
| 修 bug / 重构 / 优化 | TL | 系统化调试 + 代码质量 + 安全红线 |

## 角色说明

| 角色 | 负责 | 加载方式 |
|------|------|---------|
| **CEO** | 产品定位方法论 | `/ds:vision` 加载 |
| **PO** | 产品方案（PRD + 原型）| 自动触发（方案讨论/原型设计时） |
| **TL** | 技术交付（TDD/架构/代码质量/安全）| 自动触发（编码/调试/重构时） |

## 目录结构

```
项目根目录/
├── solution/                   # 战略、版本档案
│   ├── vision.md               # 产品宪法
│   ├── version-plan.md         # 版本演进计划（活文档）
│   ├── worklog.json            # 开发期记录（unreleased）
│   └── v{major}/               # 已发版（按 Major 分组）
│       └── v{x.y.z}/
│           ├── changelog.md    # 版本完整日志
│           └── worklog.json    # 该版本记录
├── CHANGELOG.md                # 版本索引（每版一行）
├── .claude/
│   └── plugin-state.json       # 插件配置
└── README.md
```

## FAQ

**Q: 和直接使用 openspec/superpowers 有什么区别？**

A: 本插件提供角色视角的工程规范自动注入——你正常用 openspec/superpowers 等工具，AI 自动按工作类型加载对应角色的工程要求，无需自己记住规范。

**Q: 已有项目会不会被破坏？**

A: `/ds:init` 和 `/ds:upgrade` 只新增文件和目录，不删除、不修改任何现有文件。原 CLAUDE.md 会备份为 `.bak`。两个命令都具备依赖安装能力，无论先跑哪个都能确保环境就绪。

**Q: 为什么 build 命令被移除了？**

A: v3.0 重构为三个更灵活的命令——`/ds:plan`（版本规划）、`/ds:commit`（提交记录）、`/ds:release`（发版）。日常开发不再被流程打断，工程要求通过角色自动加载自然融入。

**Q: 角色自动识别不准怎么办？**

A: 不影响正常开发。角色只是在上下文中提供工程规范，识别不准确时用户正常操作即可，不会出现错误行为。不在定义范围内的工作，插件保持透明。
