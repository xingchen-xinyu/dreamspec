---
name: ds:upgrade
description: 插件升级 — 更新 marketplace 后批量检测并升级主插件与依赖插件
---

# /ds:upgrade — 插件版本检测与升级

## 入口

用户输入 `/ds:upgrade` 触发。可在任意安装了 DreamSpec 插件的目录中独立运行。不依赖项目是否已初始化。

## 前置条件

1. 执行 `claude plugins list` 检查输出中是否包含 `dreamspec`
   - 如果不存在 → 提示用户先安装 DreamSpec 插件（安装命令详见 [PLUGINS.md](../../reference/PLUGINS.md) DreamSpec 条目），流程终止
   - 如果存在 → 从输出中提取版本号，继续流程

2. 读取 [PLUGINS.md](../../reference/PLUGINS.md) 获取各插件的 marketplace 和升级命令

## 流程

### Step 1: 更新所有 marketplace

依次刷新所有相关 marketplace（仅刷新缓存，不修改插件文件）：

```
claude plugin marketplace update dreamspec-market
claude plugin marketplace update claude-plugins-official
claude plugin marketplace update ui-ux-pro-max-skill
```

任一 marketplace 更新失败 → 不阻塞，记录并继续。

### Step 2: 汇总当前安装版本

执行 `claude plugins list` 获取已安装版本，结合 `openspec --version`，展示当前状态：

```markdown
📦 当前已安装版本

| 插件 | 当前版本 |
|------|---------|
| dreamspec | 1.3.1 |
| superpowers | 2.0.0 |
| frontend-design | 1.2.0 |
| ui_ux_max_pro | — (未安装) |
| openspec | 1.5.0 |
```

### Step 3: 用户确认

直接询问用户：

> 已更新所有 marketplace。是否尝试升级全部插件？
>
> 升级命令会自动跳过已是最新版本的插件，不会重复安装。

- 用户确认 → 进入 Step 4
- 用户跳过 → 流程结束

### Step 4: 执行升级

> **原理：** 不手动比对版本号（容易出错）。直接执行 `claude plugin update`，该命令本身就具备"已是最新则跳过"的能力。

按顺序执行升级，每个命令的输出即表明结果（已更新 / 已是最新 / 失败）：

**依赖插件：**
1. openspec：`npm install -g @fission-ai/openspec@latest` → `openspec update`
2. superpowers：`claude plugin update superpowers@claude-plugins-official --scope project`
3. frontend-design：`claude plugin update frontend-design@claude-plugins-official --scope project`
4. ui_ux_max_pro：`claude plugin marketplace update ui-ux-pro-max-skill` → `claude plugin update ui-ux-pro-max@ui-ux-pro-max-skill --scope project`

**主插件（最后执行）：**
5. dreamspec：`claude plugin update dreamspec@dreamspec-market --scope project`

**dreamspec 自升级特殊处理：**

如果 dreamspec 升级成功且用户输入包含 `--force` 关键词 → 强制升级模式，否则为增量更新：

- 增量更新（默认）：升级后检查是否需要补全目录、对比 CLAUDE.md 模板、迁移 plugin-state.json
- 强制升级（--force）：用远端最新版本覆盖本地 Skill 文件，保留用户数据

**升级失败处理：**
- 任一插件升级失败 → 不阻塞流程，记录失败信息，继续下一个
- 最终报告中标注失败项及手动修复命令

### Step 5: 汇报结果

```markdown
## /ds:upgrade 完成

**主插件：**
dreamspec vX.X.X → vY.Y.Y ✅ 已升级（或：✅ 已是最新）

**依赖插件：**
| 插件 | 结果 |
|------|------|
| superpowers | ✅ 已是最新 (vX.X.X) |
| frontend-design | ✅ 已升级 v旧 → v新 |
| ui_ux_max_pro | ⚠️ 未安装，已跳过 |
| openspec | ✅ 已是最新 (vX.X.X) |

（如有失败项，列出原因和手动修复命令）

💡 如果 dreamspec 自身已升级，可运行 `/ds:init` 同步可能新增的目录结构和配置。
```

## 强制规则

- 不手动比对版本号，依赖 `claude plugin update` 命令自身的"已是最新则跳过"能力
- 升级前先更新所有 marketplace，确保获取最新版本信息
- 升级前必须用户确认，不自动升级
- 升级失败不阻断流程，记录失败信息，继续下一个插件
- 只升级插件本身，不修改用户源代码文件（src/ 等）
