---
name: ds:init
description: 项目初始化/迁移 — 项目识别 → 项目配置 → 目录创建 → CLAUDE.md 处理 → 状态文件 → 依赖检测与安装 → 合规检测
---

# /ds:init — 项目初始化/迁移

## 入口

用户输入 `/ds:init` 触发。可在安装 DreamSpec 后直接运行——依赖缺失会在流程末尾提醒用户选择性安装。

> **提示：** 如果已通过 `/ds:upgrade` 安装过依赖，init 的依赖检测将全部显示"已就绪"，自动跳过安装步骤。

## 流程

### Step 1: 项目识别

检查项目目录状态：

- **全新项目**（空目录或无 CLAUDE.md/package.json 等工程文件）→ 默认草稿
- **已有项目**（存在 CLAUDE.md、package.json、src/ 等，但无 `.claude/plugin-state.json`）→ 迁移模式
- **已初始化项目**（存在 `.claude/plugin-state.json`）→ 重新初始化模式，沿用已有配置，补全新增目录和字段

### Step 2: 项目配置

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
- 读取已有 `plugin_version` 和 `directories` 配置
- 补全新增的标准目录和字段
- 如果用户希望修改目录配置，可在展示已有配置时确认或修正

### Step 3: 创建目录结构

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

### Step 4: CLAUDE.md 处理

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

本项目使用 DreamSpec 管理开发流程。收到开发需求时优先匹配 DreamSpec 技能
（ds:commit、ds:plan 等），除非用户显式调用其他技能。

- `/ds:vision` — 产品定位与战略
- `/ds:plan` — 版本规划（MVP、路线图）
- `/ds:commit` — 提交记录
- `/ds:release` — 发版
- `/ds:upgrade` — 插件升级

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

### Step 5: 写入状态文件

### 写入 plugin-state.json

从当前项目安装的 `.claude-plugin/plugin.json` 读取 `version` 字段获取实际插件版本号，写入 `.claude/plugin-state.json`：

```json
{
  "plugin_version": "[当前插件版本号]",
  "directories": {
    "src": ["server", "web"]
  }
}
```

### 初始化 solution/version.json

首次初始化时创建 `solution/version.json`（已存在则跳过）：

```json
{
  "currentVersion": "0.0.0",
  "versionGoal": null,
  "versionPlan": null,
  "versions": []
}
```

### Step 6: 依赖检测与安装 ⚠️非阻断

> 项目初始化（目录、CLAUDE.md、状态文件）不依赖外部插件。依赖安装为可选步骤，按需选择，安装失败不阻断流程。

#### A. 依赖检测

依赖检测采用**两级模式**：先检查设备级（全局是否安装），再检查项目级（当前项目是否已配置）。两级都满足才算"已就绪"。

**Claude Code 插件**（检查项目级是否就绪）：

执行 `claude plugins list` 获取已安装列表，然后检查项目的 `.claude/` 目录下是否有对应插件的配置目录：

| 依赖 | 用途 | 设备级检测 | 项目级检测 |
|------|------|-----------|-----------|
| superpowers | TDD + 多Agent + CodeReview + Debugging | `claude plugins list` 中是否包含 `superpowers` | `.claude/` 下是否有 superpowers 相关配置 |
| ui_ux_max_pro | HTML 原型设计 | `claude plugins list` 中是否包含 `ui-ux-pro-max` | `.claude/` 下是否有 ui-ux-pro-max 相关配置 |
| frontend-design | 前端 UI 组件设计与生成 | `claude plugins list` 中是否包含 `frontend-design` | `.claude/` 下是否有 frontend-design 相关配置 |

> **判定规则：** 设备级满足 + 项目级满足 → ✅ 已就绪。设备级满足但项目级缺失 → ⚠️ 需安装项目级。设备级缺失 → ❌ 需完整安装。

**openspec**（检查项目级是否就绪）：

| 依赖 | 用途 | 设备级检测 | 项目级检测 |
|------|------|-----------|-----------|
| openspec | Spec 编写 + Tasks 管理 | `openspec --version` 是否可用 | 项目根目录下 `openspec/` 目录是否存在 |

> **判定规则：** 设备级（全局 npm 包）+ 项目级（`openspec/` 目录）两级都满足才算"已就绪"。**openspec 全局安装 ≠ 当前项目已配置**——每个项目必须独立执行 `openspec init` 生成 `openspec/` 目录。

#### B. 输出检测结果

检测完成后，结合 `claude plugins list` 和 `openspec --version` 的输出，汇总展示依赖状态和版本号：

```markdown
📋 依赖检测结果

| 依赖 | 设备级 | 项目级 | 版本 | 状态 |
|------|--------|--------|------|------|
| superpowers | ✅ | ✅ | 2.0.0 | 已就绪 |
| frontend-design | ✅ | ⚠️ 缺失 | 1.2.0 | 需项目级安装 |
| ui_ux_max_pro | ❌ 未安装 | ❌ 未安装 | — | 需完整安装 |
| openspec | ✅ | ❌ 缺失 | 1.5.0 | 需项目级配置 |
```

#### C. 用户选择

**所有依赖已就绪** → 跳过安装，直接进入 Step 7。

**有缺失** → 使用 AskUserQuestion 让用户勾选要安装的依赖（多选，默认全选）：

```json
{
  "questions": [{
    "question": "以下依赖未就绪，选择要安装的依赖（可多选，全不选则跳过）：",
    "header": "选择依赖",
    "multiSelect": true,
    "options": [
      {"label": "ui_ux_max_pro", "description": "需完整安装（marketplace + install）"},
      {"label": "openspec", "description": "需项目级配置（openspec init + config profile + update）"},
      {"label": "frontend-design", "description": "需项目级安装（install --scope project）"}
    ]
  }]
}
```

- 用户全不选 → 跳过安装，提示后续可通过 `/ds:upgrade` 补装
- 用户选择部分 → 只安装勾选的依赖

#### D. 按需安装（只安装用户勾选的依赖）

| 依赖 | 缺失情况 | 需执行的步骤 |
|------|---------|-------------|
| **openspec** | 仅项目级缺失（全局已安装） | ② `cd <项目根目录>` ③ `openspec init` ④ `openspec config profile`（选择 expanded workflows 扩展模式） ⑤ `openspec update` |
| **openspec** | 设备级 + 项目级都缺失 | ① `npm install -g @fission-ai/openspec@latest` ② `cd <项目根目录>` ③ `openspec init` ④ `openspec config profile` ⑤ `openspec update` |
| **superpowers** / **frontend-design** | 任意级别缺失 | `claude plugins install <plugin>@claude-plugins-official --scope project`（幂等，已安装则跳过） |
| **ui_ux_max_pro** | 任意级别缺失 | ① `claude plugins marketplace add nextlevelbuilder/ui-ux-pro-max-skill` ② `claude plugins install ui-ux-pro-max@ui-ux-pro-max-skill --scope project`（幂等） |

> **红线：** openspec 的项目级配置（步骤 ②③④⑤）是**最容易遗漏**的环节——设备全局已安装 openspec 时，AI 容易错误地判定为"已就绪"而跳过。必须检查 `openspec/` 目录是否存在，不存在就强制执行项目级配置。openspec config profile 和 update 是启用扩展模式（explore → propose → apply）的必要步骤，缺一不可。

#### E. 安装后验证

每个依赖安装完成后，立即验证是否真正可用：
- Claude Code 插件：执行 `claude plugins list` 确认插件出现在列表中；
- openspec：执行 `openspec --version` 确认命令可用；**并且**检查项目目录下 `openspec/` 目录是否已生成

#### F. 安装结果
- ✅ 全部安装成功 → 进入 Step 7
- ⚠️ 部分/全部安装失败 → **记录警告**，展示失败原因和手动安装命令，继续 Step 7（不阻断）
- ⏭️ 用户选择跳过 → 提示后续可通过 `/ds:upgrade` 补装，进入 Step 7

### Step 7: 项目标准合规检测

> 检测结果为**警告**而非阻断——不合规项给出修复建议，但不阻止后续流程。

对项目进行合规扫描：

```
📋 项目合规检测

文件存在性：
  ✅/⚠️ CLAUDE.md
  ✅/⚠️ .claude/plugin-state.json
  ✅/⚠️ README.md
  ✅/⚠️ solution/version.json

plugin-state.json 字段完整性：
  ✅/⚠️ plugin_version 已记录
  ✅/⚠️ directories 已配置

目录结构：
  ✅/⚠️ solution/ 目录存在
  ✅/⚠️ src/server/ 目录存在
  ✅/⚠️ src/web/ 目录存在

CLAUDE.md 结构：
  ✅/⚠️ "## 项目" 章节存在
  ✅/⚠️ "## 仓库结构" 章节存在
  ✅/⚠️ "## 工作方式" 章节存在
  ✅/⚠️ 包含 /ds:vision、/ds:plan、/ds:commit、/ds:release 命令引用
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
（如用户选择跳过，标注 ⏭️ 及补装提示）

**合规检测：** [✅ 全部通过 / ⚠️ X 项警告（列出修复建议）]

**目录结构：** [新增的目录列表]
**CLAUDE.md：** [处理结果]

**下一步：**
- 执行 `/ds:vision` 梳理产品定位（完成后自动回填项目信息），待 vision 完成后开始描述你的需求（build 自动触发）
```

## 强制规则

- Step 6 依赖检测与安装为可选步骤：检测缺失后由用户勾选安装，安装失败不阻断流程，项目初始化本身不依赖外部插件
- 只新增文件和目录，不删除、不修改用户源代码文件（src/ 等），CLAUDE.md 和 plugin-state.json 在升级时需用户确认后更新
- CLAUDE.md 必须备份原文件为 `.bak` 后才精简
- 每次只问一个问题
- 已初始化项目可重复运行 init，流程幂等：只补全新增内容，不覆盖已有配置
- Step 7 合规检测为警告级别，不阻断流程
