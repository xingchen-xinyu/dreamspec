---
name: ds:init
description: 项目初始化/迁移 — 依赖就绪保障 → 项目配置 → 目录创建 → CLAUDE.md 处理 → 合规检测
---

# /ds:init — 项目初始化/迁移

## 入口

用户输入 `/ds:init` 触发。可在安装 DreamSpec 后直接运行——依赖缺失会自动安装。

> **提示：** 如果已通过 `/ds:upgrade` 安装过依赖，init 会自动跳过依赖安装步骤。

## 流程

### Step 1: 项目识别

检查项目目录状态：

- **全新项目**（空目录或无 CLAUDE.md/package.json 等工程文件）→ 默认草稿
- **已有项目**（存在 CLAUDE.md、package.json、src/ 等，但无 `.claude/plugin-state.json`）→ 迁移模式
- **已初始化项目**（存在 `.claude/plugin-state.json`）→ 重新初始化模式，沿用已有配置，补全新增目录和字段

### Step 2: 依赖检测与安装 🚫阻断

> **红线：依赖是后续流程的前置条件，所有依赖必须安装成功后才能继续。安装失败则流程终止，用户解决问题后重新执行。**

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

> **💡 提示：** 如果已通过 `/ds:upgrade` 安装过依赖，此步骤所有检测项将全部跳过，直接进入 Step 3。

### Step 3: 项目配置

**全新项目 — 默认草稿：**

初次初始化全部采用默认值，零交互。待 /ds:vision 阶段完成后回填最终信息。

默认配置：
- 项目名称：当前根目录名称
- 一句话简介：留空（/ds:vision 阶段完成后回填）
- 技术栈：留空（/ds:vision 阶段完成后确定）
- 目录结构：默认（server + web）

**已有项目 — 自动识别 + 确认：**

1. 自动识别技术栈（从 package.json、目录结构等推断）
2. 自动识别现有版本（从 openspec、git tag 等推断）
3. 展示识别结果，等待用户确认或修正

**已初始化项目**（存在 `.claude/plugin-state.json`）：
- 读取已有配置，沿用现有 `plugin-state.json` 中的项目信息
- 仅更新可能变化的字段（如 `installed_at`）
- 如果用户希望修改项目配置，可在展示已有配置时确认或修正

### Step 4: 创建目录结构

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

### Step 5: CLAUDE.md 处理

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

战略调整：/ds:vision
新功能开发：/ds:build → Explore → Spec → Demo → Build
Bug 修复：/ds:fix

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

### Step 6: 写入状态文件

从当前项目安装的 `.claude-plugin/plugin.json` 读取 `version` 字段获取实际插件版本号，写入 `.claude/plugin-state.json`：

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
    "vision_completed": false
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

### Step 7: 项目标准合规检测

> 检测结果为**警告**而非阻断——不合规项给出修复建议，但不阻止后续流程。

对项目进行合规扫描：

```
📋 项目合规检测

文件存在性：
  ✅/⚠️ CLAUDE.md
  ✅/⚠️ .claude/plugin-state.json
  ✅/⚠️ README.md

plugin-state.json 字段完整性：
  ✅/⚠️ project.name 已填写
  ✅/⚠️ plugin_version 已记录
  ✅/⚠️ phases.init_completed = true
  ✅/⚠️ techStack 结构完整（frontend/backend/platforms）
  ✅/⚠️ directories 已配置

目录结构：
  ✅/⚠️ solution/ 目录存在
  ✅/⚠️ src/server/ 目录存在
  ✅/⚠️ src/web/ 目录存在

CLAUDE.md 结构：
  ✅/⚠️ "## 项目" 章节存在
  ✅/⚠️ "## 仓库结构" 章节存在
  ✅/⚠️ "## 工作方式" 章节存在
  ✅/⚠️ 包含 /ds:vision、/ds:build、/ds:fix 命令引用
```

### Step 8: 汇报结果

```markdown
## /ds:init 完成

**项目类型：** [全新/已有/重新初始化]
**插件版本：** dreamspec vX.X.X

**依赖状态：**
✅ superpowers（已安装，vX.X.X）
✅ frontend-design（已安装，vX.X.X）
✅ ui_ux_max_pro（已安装，vX.X.X）
✅ openspec（已安装，vX.X.X）

（如有依赖安装失败，在此标注 ❌ 及手动安装命令）

**合规检测：** [✅ 全部通过 / ⚠️ X 项警告（列出修复建议）]

**目录结构：** [新增的目录列表]
**CLAUDE.md：** [处理结果]

**下一步：**
- 执行 `/ds:vision` 制定产品愿景（完成后自动回填项目信息），待 vision 完成后再 /ds:build（开始版本交付）
```

## 强制规则

- Step 2 依赖检测为🚫阻断步骤：缺失依赖自动安装，安装失败流程终止，用户解决问题后重新执行
- 只新增文件和目录，不删除、不修改用户源代码文件（src/ 等），CLAUDE.md 和 plugin-state.json 在升级时需用户确认后更新
- CLAUDE.md 必须备份原文件为 `.bak` 后才精简
- 每次只问一个问题
- 已初始化项目可重复运行 init，流程幂等：只补全新增内容，不覆盖已有配置
- Step 7 合规检测为警告级别，不阻断流程
