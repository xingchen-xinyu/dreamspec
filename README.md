# AI Team — AI Agent 创业开发团队

将 openspec、superpowers、ui_ux_max_pro 等技能的开发流程和角色协作方法固化为一个 Claude Code 插件。通过 CEO/PO/TL/QA 四个 AI 角色按标准流程协作，帮你把想法从战略规划到编码交付完整落地。

## 安装

```bash
# Step 1: 注册 marketplace
claude plugin marketplace add xingchen-xinyu/dreamspec --scope project
# Step 2: 安装插件
claude plugin install dreamspec@dreamspec-market --scope project
```

安装后自动注册以下命令：`/ds:init`  `/ds:upgrade`  `/ds:vision`  `/ds:build`  `/ds:fix`

依赖自动安装：openspec + superpowers + ui_ux_max_pro

## 升级

升级分为两步：更新插件包 + 同步项目文件。DreamSpec 提供 `/ds:upgrade` 命令一站式完成。

### 使用 /ds:upgrade（推荐）

在 Claude Code 中运行：

```
/ds:upgrade       # 检测主插件和依赖插件版本，用户确认后执行升级
```

`/ds:upgrade` 会：
1. 检测 dreamspec 主插件是否有新版本（从 GitHub 获取远端版本号）
2. 检测所有依赖插件（superpowers、frontend-design、ui_ux_max_pro、openspec）是否有更新
3. 展示版本差异汇总表，等待你确认
4. 依次升级依赖插件，最后升级 dreamspec 自身（含项目目录和配置同步）

升级完成后会提示可运行 `/ds:init` 同步可能新增的目录结构和配置。

### 强制升级

如果升级过程中出现问题，或需要强制覆盖本地文件：

```
/ds:upgrade --force      # 强制升级，用远端最新版本覆盖本地 Skill 文件，保留用户数据
```

### 手动升级（备选）

```bash
# 更新 marketplace 和插件到最新版本
claude plugin marketplace update dreamspec-market
claude plugin update dreamspec@dreamspec-market --scope project
```

然后在项目目录中运行 `/ds:init` 同步目录结构和配置。

## 卸载

```bash
claude plugin uninstall dreamspec@dreamspec-market
```

## 快速开始

### 全新项目

```bash
mkdir my-app && cd my-app
claude plugin marketplace add xingchen-xinyu/dreamspec --scope project
claude plugin install dreamspec@dreamspec-market --scope project
```

在 Claude Code 中：

```
/ds:init          # 配置项目信息，创建目录结构
/ds:vision      # 定位产品、规划版本路线
/ds:build         # 按版本迭代交付（Explore → Spec → Demo → Build）
```

### 已有项目

```bash
cd existing-project
claude plugin marketplace add xingchen-xinyu/dreamspec --scope project
claude plugin install dreamspec@dreamspec-market --scope project
```

在 Claude Code 中：

```
/ds:init          # 检测项目状态，无痛迁移，不删除任何现有文件
/ds:build         # 继续当前版本的开发
```

## 命令参考

| 命令 | 用途 | 何时用 |
|------|------|--------|
| `/ds:init` | 项目初始化/迁移 | 新项目开始、切换到插件模式 |
| `/ds:upgrade` | 插件版本检测与升级 | 检查并升级主插件及依赖插件，支持 `--force` 强制覆盖 |
| `/ds:vision` | 产品愿景 | 0→1 必须，后续按需调整战略 |
| `/ds:build` | 版本交付 | 每个版本迭代 |
| `/ds:fix` | 问题修复 | Bug 修复 |

## 工作流

### /ds:build 四步

```
注入角色上下文 → openspec(Explore + Spec) → Demo → Build
                    ↓                          ↓       ↓
              specification.md              原型确认  编码交付
                 (= PRD)                   (ui_ux)   (openspec
                                                    + superpowers)
```

每步完成后用户确认，QA 贯穿全流程。

## 角色说明

| 角色 | 负责 | 参与场景 |
|------|------|---------|
| **CEO** | 商业价值、战略方向 | `/ds:vision`（主导）、`/ds:build` Explore |
| **PO** | 产品设计、Spec、原型 | `/ds:build` Spec + Demo（主导） |
| **TL** | 技术方案、编码交付 | `/ds:build` Build（主导）、`/ds:fix`（主导） |
| **QA** | 质量审查、规范检查 | 全流程贯穿 |

## 目录结构

```
项目根目录/
├── solution/          # 战略、Spec、原型
│   ├── vision.md    # 产品愿景
│   └── v{x}/
│       ├── specification.md  # PRD（openspec 输出）
│       ├── tech-design.md    # 技术方案
│       └── demo/             # HTML 原型
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

A: `/ds:init` 和 `/ds:upgrade` 只新增文件和目录，不删除、不修改任何现有文件。原 CLAUDE.md 会备份为 `.bak`。

**Q: 每个阶段必须走完吗？**

A: 每步完成后你可以选择跳过或调整。流程是骨架，你是决策者。

**Q: 无前端的后端项目 Demo 步骤怎么处理？**

A: `/ds:build` Explore 阶段可以声明跳过 Demo，纯后端项目可省略。
