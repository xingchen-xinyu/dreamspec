---
name: ds:init
description: 项目初始化/迁移/升级 — 创建目录结构、检查依赖、精简 CLAUDE.md、检测版本升级
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

### Step 2: 项目配置

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

**插件升级 — 自动检测 + 增量/强制更新：**

1. 读取 `.claude/plugin-state.json` 中的 `plugin_version`，与当前 `plugin.json` 的 `version` 对比
2. 版本相同时：
   a. 检查用户输入是否包含「强制升级」或 `--force` 关键词
      - 是 → 跳过确认，直接执行强制升级（跳至步骤 4）
   b. 未检测到强制关键词 → 询问用户："当前已是最新版本 (vX.X.X)。是否需要强制用远端最新版本覆盖本地插件？"
      - 用户选择是 → 执行强制升级（跳至步骤 4）
      - 用户选择否 → 提示"已是最新版"，直接结束
3. 版本不同时：
   a. 检查用户输入是否包含「强制升级」或 `--force` 关键词
      - 是 → 跳过增量更新，直接执行强制升级（跳至步骤 4）
   b. 未检测到强制关键词 → 执行增量更新：
      - 目录补全：新增不存在的目录（如新版本新增的目录结构），不删现有文件
      - CLAUDE.md 对比：读取当前 CLAUDE.md，与新版模板对比，提示变更内容，等用户确认后更新
      - plugin-state.json 迁移：补全新版中新增的字段（不删旧字段），更新 `plugin_version`
      - 跳至步骤 5
4. **强制升级**（无论版本是否一致，用远端最新版本覆盖本地插件）：
   a. 执行 marketplace 更新：`claude plugin marketplace update dreamspec-market`
   b. 执行插件更新：`claude plugin update dreamspec@dreamspec-market`（远端最新版本覆盖本地 Skill 文件）
   c. 目录检查：确保新版本所需目录结构齐全（只新增不删除）
   d. CLAUDE.md 同步：以本 Skill 的 Step 5 中定义的 CLAUDE.md 骨架模板作为对比基准，与本地 CLAUDE.md 对比，保留用户已填写的自定义信息（项目名称、描述、技术栈），更新框架内容，用户确认后写入
   e. plugin-state.json 迁移：补全新版中新增的字段，**保留**用户已填写的信息（project.name、project.description、techStack、directories、currentVersion 等），更新 `plugin_version`
5. 汇报升级结果：版本变更（X.X.X → Y.Y.Y）+ 更新了哪些文件

### Step 3: 创建目录结构

只新增，不删除现有文件/目录：

```
项目根目录/
├── solution/            # 新增（如果不存在）
├── src/
│   ├── server/          # 新增（如果不存在）
│   └── web/             # 新增（如果不存在）
├── tests/               # 新增（如果不存在）
├── .claude/             # 新增（如果不存在）
│   └── plugin-state.json  # 新增（写入状态模板）
└── README.md            # 新增（如果不存在）
```

### Step 4: 依赖检查

依赖分为两类，分别检查。详细安装命令参考 [PLUGINS.md](../../reference/PLUGINS.md)。

**A. Claude Code 插件**（检查是否已安装）：

```
claude plugins list
```

| 依赖 | 用途 | 检查方式 |
|------|------|---------|
| superpowers | TDD + 多Agent + CodeReview + Debugging | `claude plugins list` 中是否包含 `superpowers` |
| ui_ux_max_pro | HTML 原型设计 | `claude plugins list` 中是否包含 `ui-ux-pro-max` |
| frontend-design | 前端 UI 组件设计与生成 | `claude plugins list` 中是否包含 `frontend-design` |

**B. npm 全局包**（检查是否已安装）：

| 依赖 | 用途 | 检查方式 |
|------|------|---------|
| openspec | Spec 编写 + Tasks 管理 | `openspec --version` 或 `npm list -g @fission-ai/openspec` |

**C. 汇报检查结果：**

列出检查结果：已安装（✅）/ 缺失（❌），缺失时从 [PLUGINS.md](../../reference/PLUGINS.md) 获取对应的安装命令提示用户。

所有依赖默认安装到**项目级**（Claude Code 插件加 `--scope project`，npm 包在项目目录执行 `openspec init`）。

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

### Step 6: 写入状态文件

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

### Step 7: 汇报结果

```markdown
## /ds:init 完成

**项目类型：** [全新/已有]
**目录结构：** [新增的目录列表]
**依赖状态：** [各依赖检查结果]
**CLAUDE.md：** [处理结果]

**下一步：** /strategy（制定产品战略，完成后自动回填项目信息），待 strategy 完成后再 /build（开始版本交付）
```

## 强制规则

- 只新增文件和目录，不删除、不修改用户源代码文件（src/、tests/ 等），CLAUDE.md 和 plugin-state.json 在升级时需用户确认后更新
- CLAUDE.md 必须备份原文件为 `.bak` 后才精简
- 每次只问一个问题
- 依赖缺失不阻塞，给出安装命令即可
