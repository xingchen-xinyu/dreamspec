---
name: ds:init
description: 项目初始化/迁移/升级 — 依赖检查与强制安装 → 插件版本检测 → 项目配置 → 目录创建 → CLAUDE.md 处理
---

# /ds:init — 项目初始化/迁移/升级

## 入口

用户输入 `/ds:init` 触发。

## 流程

### Step 1: 项目识别

检查项目目录状态：

- **全新项目**（空目录或无 CLAUDE.md/package.json 等工程文件）→ 默认草稿
- **已有项目**（存在 CLAUDE.md、package.json、src/ 等）→ 迁移模式
- **插件升级**（存在 `.claude/plugin-state.json`）→ 升级模式

### Step 2: 依赖检查与强制安装 🚫阻断

> **红线：所有依赖必须安装成功后才能继续下一步。安装失败则流程终止，用户解决问题后重新执行。**

依赖分为两类，分别检查。详细安装命令参考 [PLUGINS.md](../../reference/PLUGINS.md)。

**A. Claude Code 插件**（检查是否已安装）：

执行 `claude plugins list` 检查：

| 依赖 | 用途 | 检查方式 |
|------|------|---------|
| superpowers | TDD + 多Agent + CodeReview + Debugging | `claude plugins list` 中是否包含 `superpowers` |
| ui_ux_max_pro | HTML 原型设计 | `claude plugins list` 中是否包含 `ui-ux-pro-max` |
| frontend-design | 前端 UI 组件设计与生成 | `claude plugins list` 中是否包含 `frontend-design` |

**B. npm 全局包**（检查是否已安装）：

| 依赖 | 用途 | 检查方式 |
|------|------|---------|
| openspec | Spec 编写 + Tasks 管理 | `openspec --version` 或 `npm list -g @fission-ai/openspec` |

**C. 自动安装缺失依赖：**

1. 汇总所有缺失的依赖
2. 对每个缺失依赖，从 [PLUGINS.md](../../reference/PLUGINS.md) 获取安装命令并**自动执行安装**
3. 所有依赖默认安装到**项目级**（Claude Code 插件加 `--scope project`，npm 包在项目目录执行 `openspec init`）
4. 安装结果：
   - ✅ 全部安装成功 → 进入 Step 3
   - ❌ 任一安装失败 → **流程终止**，展示失败原因和手动安装命令，提示用户解决问题后重新执行 `/ds:init`

### Step 3: 插件版本检测与升级

检查主插件和依赖插件是否有新版本，由用户决定是否升级。

**A. 主插件（dreamspec）版本检测：**

1. 更新 marketplace：`claude plugin marketplace update dreamspec-market`
2. 读取 `.claude/plugin-state.json` 中的 `plugin_version`（全新项目无此字段，视为首次安装）
3. 读取当前插件 `plugin.json` 的 `version` 作为最新可用版本
4. 对比：
   - 首次安装 → 提示"首次安装 dreamspec vX.X.X"
   - 版本相同 → 提示"dreamspec 已是最新版本 (vX.X.X)"
   - 有新版本 → 提示"dreamspec 可升级：v旧 → v新"

**B. 依赖插件版本检测：**

对每个已安装的依赖插件，检查是否有新版本可用（仅检测，不执行升级，升级在步骤 D 用户确认后执行）：

1. 执行 `claude plugins list` 获取每个依赖插件的当前安装版本
2. 对比 marketplace 上的最新版本（通过 marketplace 信息或检查更新提示）
3. 汇总版本差异

**C. 汇总版本检测结果：**

```markdown
📦 插件版本检测

| 插件 | 当前版本 | 最新版本 | 状态 |
|------|---------|---------|------|
| dreamspec | 1.0.0 | 1.1.0 | 🔄 可升级 |
| superpowers | 2.0.0 | 2.0.0 | ✅ 已是最新 |
| frontend-design | 1.0.0 | 1.2.0 | 🔄 可升级 |

共 2 个插件可升级。是否升级到最新版本？
```

**D. 用户决策：**

- 用户选择升级 → 依次执行升级命令（marketplace update + plugin update --scope project）
- 用户选择跳过 → 保持当前版本，继续下一步
- 升级失败 → 不阻塞流程，记录失败信息，在最终报告中提示

**E. 插件升级（dreamspec 自身）的特殊处理：**

当 dreamspec 自身需要升级时，执行增量/强制更新（与旧版升级逻辑一致）：

1. 检查用户输入是否包含「强制升级」或 `--force` 关键词 → 强制升级
2. 版本不同且非强制 → 执行增量更新：
   - 目录补全：新增不存在的目录，不删现有文件
   - CLAUDE.md 对比：读取当前 CLAUDE.md，与新版模板对比，等用户确认后更新
   - plugin-state.json 迁移：补全新字段，保留用户数据，更新 `plugin_version`
3. 强制升级 → 用远端最新版本覆盖本地 Skill 文件，保留用户数据
4. 汇报升级结果

### Step 4: 项目配置

**全新项目 — 默认草稿：**

初次初始化全部采用默认值，零交互。待 /strategy 阶段完成后回填最终信息。

默认配置：
- 项目名称：当前根目录名称
- 一句话简介：留空（/strategy 阶段完成后回填）
- 技术栈：留空（/strategy 阶段完成后确定）
- 目录结构：默认（server + web）

**已有项目 — 自动识别 + 确认：**

1. 自动识别技术栈（从 package.json、目录结构等推断）
2. 自动识别现有版本（从 openspec、git tag 等推断）
3. 展示识别结果，等待用户确认或修正

**插件升级模式**（存在 `.claude/plugin-state.json`）：
- Step 2/3 已处理依赖和版本检测
- 如果 Step 3 执行了插件升级，本步骤跳过项目配置（沿用已有配置）
- 如果是纯升级场景（非首次 init），升级完成后流程结束

### Step 5: 创建目录结构

只新增，不删除现有文件/目录：

```
项目根目录/
├── solution/            # 新增（如果不存在）
├── src/
│   ├── server/          # 新增（如果不存在）
│   └── web/             # 新增（如果不存在）
├── .claude/             # 新增（如果不存在）
│   └── plugin-state.json  # 新增（写入状态模板）
└── README.md            # 新增（如果不存在）
```

### Step 6: CLAUDE.md 处理

**全新项目：**

生成轻量 CLAUDE.md 骨架（覆盖写入）：

```markdown
# CLAUDE.md

## 项目

{{PROJECT_INFO}}

## 仓库结构

src/server/  # 后端
src/web/     # 前端

## 技术栈

{{TECH_STACK}}

## 工作方式

战略调整：/strategy
新功能开发：/build → Explore → Spec → Demo → Build
Bug 修复：/fix

```

**已有项目 — 递归处理所有 CLAUDE.md：**

1. 扫描所有子目录中的 CLAUDE.md：`find . -name "CLAUDE.md" -not -path "*/node_modules/*" -not -path "*/.git/*"`
2. 对每个文件：
   a. 备份原文件：`CLAUDE.md` → `CLAUDE.md.bak`
   b. 提取关键信息：项目定位、技术栈、仓库结构
   c. 移除：编码纪律、规范细节、TDD 流程（已内化到角色 Skill）
   d. 保留：项目名称、简介、技术栈、目录结构、常用命令
   e. 写入精简后的 CLAUDE.md
3. 汇报处理结果（处理了哪些文件，精简了多少内容）

### Step 7: 写入状态文件

写入 `.claude/plugin-state.json`：

```json
{
  "project": {
    "name": "[根目录名称]",
    "description": ""
  },
  "plugin_version": "[当前插件版本号]",
  "currentVersion": "0.0.0",
  "status": "init_completed",
  "phases": {
    "init_completed": true,
    "strategy_completed": false
  },
  "techStack": {
    "frontend": [],
    "backend": [],
    "platforms": []
  },
  "directories": {
    "src": ["server", "web"]
  },
  "installed_at": "[当前日期]"
}
```

### Step 8: 汇报结果

```markdown
## /ds:init 完成

**项目类型：** [全新/已有/升级]
**插件版本：** dreamspec vX.X.X（[首次安装/已是最新/已升级: v旧→v新]）

**依赖状态：**
✅ superpowers（已安装，vX.X.X）
✅ frontend-design（已安装，vX.X.X）
✅ ui_ux_max_pro（已安装，vX.X.X）
✅ openspec（已安装，vX.X.X）

（如有依赖插件在 Step 3 升级了，标注：🔄 superpowers 已升级 v旧→v新）

**目录结构：** [新增的目录列表]
**CLAUDE.md：** [处理结果]

**下一步：** /strategy（制定产品战略，完成后自动回填项目信息），待 strategy 完成后再 /build（开始版本交付）
```

## 强制规则

- Step 2 依赖检查为🚫阻断步骤：缺失依赖自动安装，安装失败流程终止，用户解决问题后重新执行
- Step 3 插件版本检测：汇总主插件和依赖插件的版本信息，由用户决定是否升级，不自动升级
- 只新增文件和目录，不删除、不修改用户源代码文件（src/ 等），CLAUDE.md 和 plugin-state.json 在升级时需用户确认后更新
- CLAUDE.md 必须备份原文件为 `.bak` 后才精简
- 每次只问一个问题
