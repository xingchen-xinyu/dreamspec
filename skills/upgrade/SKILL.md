---
name: ds:upgrade
description: 插件升级 — 主插件与依赖插件版本检测，用户确认后执行升级
---

# /ds:upgrade — 插件版本检测与升级

## 入口

用户输入 `/ds:upgrade` 触发。可在任意安装了 DreamSpec 插件的目录中独立运行。不依赖项目是否已初始化。

## 前置条件

1. 确认 DreamSpec 插件已安装：
   - 检查 `.claude-plugin/plugin.json` 是否存在（插件安装时自带此文件）
   - 如果不存在 → 提示用户先通过 marketplace 安装 DreamSpec 插件（安装命令详见 [PLUGINS.md](../../reference/PLUGINS.md) DreamSpec 条目），流程终止
   - 如果存在 → 读取 `version` 获取当前安装的插件版本，继续流程

2. 读取 [PLUGINS.md](../../reference/PLUGINS.md) 获取各插件的升级命令

## 流程

### Step 1: 主插件（dreamspec）版本检测

1. 检查 marketplace 是否已注册：如果尚未注册 `dreamspec-market`，提示用户先注册 marketplace 并安装插件（安装命令详见 [PLUGINS.md](../../reference/PLUGINS.md) DreamSpec 条目）
2. 更新 marketplace：执行 `claude plugin marketplace update dreamspec-market`
3. 当前安装版本：已在前置条件中从 `.claude-plugin/plugin.json` 读取 `version` 字段。如果项目已初始化且 `.claude/plugin-state.json` 中存在 `plugin_version` 字段，以 state.json 中的记录为准
4. 获取远端最新版本（不可读本地缓存，本地缓存可能未同步）：
   - 读取当前项目安装的 `.claude-plugin/plugin.json`，从中提取 `repository` 字段获取 GitHub 仓库地址（如 `https://github.com/xingchen-xinyu/dreamspec`）
   - 拼接 raw URL：`https://raw.githubusercontent.com/<owner>/<repo>/main/.claude-plugin/plugin.json`
   - Fetch 该 URL，从返回的 JSON 中读取 `version` 作为远端最新版本
5. 对比：
   - 版本相同 → 提示"dreamspec 已是最新版本 (vX.X.X)"
   - 有新版本 → 提示"dreamspec 可升级：v旧 → v新"

### Step 2: 依赖插件版本检测

对每个已安装的依赖插件，检查是否有新版本可用（仅检测，不执行升级，升级在 Step 5 用户确认后执行）：

1. 执行 `claude plugins list` 获取每个依赖插件的当前安装版本
2. 对每个已安装的依赖插件，执行 `claude plugin search <插件名>` 查看 marketplace 上的最新版本号
3. 对 openspec（npm 全局包），执行 `npm view @fission-ai/openspec version` 对比 `openspec --version`
4. 汇总版本差异

> 注意：如果某个依赖插件未安装（不在 `claude plugins list` 中），标记为"未安装"，不阻塞流程，在汇总中提示用户可通过 `/ds:init` 安装。

### Step 3: 汇总版本检测结果

```markdown
📦 插件版本检测

| 插件 | 当前版本 | 最新版本 | 状态 |
|------|---------|---------|------|
| dreamspec | 1.0.0 | 1.1.0 | 🔄 可升级 |
| superpowers | 2.0.0 | 2.0.0 | ✅ 已是最新 |
| frontend-design | 1.0.0 | 1.2.0 | 🔄 可升级 |
| ui_ux_max_pro | — | 1.5.0 | ⚠️ 未安装 |
| openspec | 1.0.0 | 1.1.0 | 🔄 可升级 |

共 N 个插件可升级。是否升级到最新版本？
```

### Step 4: 用户决策

- 用户选择升级全部 → 进入 Step 5 执行所有可升级插件的升级
- 用户选择跳过 → 保持当前版本，流程结束
- 用户选择部分升级 → 按用户指定的插件列表执行

### Step 5: 执行升级

对于每个用户确认要升级的插件，逐一从 [PLUGINS.md](../../reference/PLUGINS.md) 获取各依赖插件的「升级命令」并按顺序执行：

**依赖插件升级顺序：**
1. openspec
2. superpowers
3. frontend-design
4. ui_ux_max_pro

**主插件（dreamspec）特殊处理 — 最后执行：**

当 dreamspec 自身需要升级时：

1. 检查用户输入是否包含「强制升级」或 `--force` 关键词 → 强制升级模式
2. 版本不同且非强制 → 执行增量更新：
   - 目录补全：新增不存在的目录，不删现有文件
   - CLAUDE.md 对比：读取当前 CLAUDE.md，与新版模板对比，展示差异后等用户确认再更新
   - 如果存在 `.claude/plugin-state.json`：补全新字段，保留用户数据，更新 `plugin_version`
3. 强制升级 → 用远端最新版本覆盖本地 Skill 文件，保留用户数据
4. 汇报升级结果

**升级失败处理：**
- 任一插件升级失败 → 不阻塞流程，记录失败信息，继续升级其他插件
- 所有升级完成后在最终报告中标注失败项及手动修复命令
- 升级 dreamspec 自身失败 → 保留旧版本可用，记录失败原因

**自升级后说明：**
- 升级 dreamspec 自身会修改 Skill 文件（包括本 upgrade SKILL.md），升级完成后提示用户 Skill 已更新，建议查看最新文档

### Step 6: 汇报结果

```markdown
## /ds:upgrade 完成

**主插件：** dreamspec vX.X.X（[已是最新/已升级: v旧→v新]）

**依赖插件：**
| 插件 | 操作 | 结果 |
|------|------|------|
| superpowers | 升级 v旧→v新 | ✅ 成功 |
| frontend-design | 已是最新 vX.X.X | — |
| ui_ux_max_pro | （未安装，已跳过） | ⚠️ 可通过 /ds:init 安装 |
| openspec | 升级 v旧→v新 | ✅ 成功 |

（如有升级失败项，在此列出失败原因和手动修复命令）
```

## 强制规则

- 升级前版本检测以 `.claude-plugin/plugin.json` 为准，`plugin-state.json` 为辅助记录
- 版本检测阶段除 marketplace 缓存刷新外不修改 Skill 文件或用户文件，仅读取和对比
- 依赖插件缺失时不阻断流程，标记为"未安装"，提示通过 `/ds:init` 安装
- 升级前必须用户确认，不自动升级
- 升级失败不阻断流程，记录失败信息在最终报告中提供手动命令
- 只升级插件本身，不修改用户源代码文件（src/ 等）
- 插件升级完成后，必须提示用户可运行 `/ds:init` 同步可能新增的目录结构和配置
