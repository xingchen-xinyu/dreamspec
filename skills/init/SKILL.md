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

- **全新项目**（空目录或无 CLAUDE.md/package.json 等工程文件）→ 交互式配置
- **已有项目**（存在 CLAUDE.md、package.json、src/ 等）→ 迁移模式
- **插件升级**（存在 `.claude/plugin-state.json`）→ 升级模式

### Step 2: 项目配置

**全新项目 — 交互式确认：**

每次只问一个问题：
1. 项目名称是什么？
2. 一句话简介？
3. 技术栈偏好（前端/后端/平台）？可以等 /strategy 阶段再确定
4. 是否有特殊目录需求？（默认 server + web）

**已有项目 — 自动识别 + 确认：**

1. 自动识别技术栈（从 package.json、目录结构等推断）
2. 自动识别现有版本（从 openspec、git tag 等推断）
3. 展示识别结果，等待用户确认或修正

**插件升级 — 自动检测 + 增量更新：**

1. 读取 `.claude/plugin-state.json` 中的 `plugin_version`，与当前 `plugin.json` 的 `version` 对比
2. 版本相同时 → 提示"已是最新版"，直接结束
3. 版本不同时，执行增量更新：
   - 目录补全：新增不存在的目录（如新版本新增的目录结构），不删现有文件
   - CLAUDE.md 对比：读取当前 CLAUDE.md，与新版模板对比，提示变更内容，等用户确认后更新
   - plugin-state.json 迁移：补全新版中新增的字段（不删旧字段），更新 `plugin_version`
4. 汇报升级结果：版本变更（1.0.0 → 1.1.0）+ 更新了哪些文件

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

[项目名称] — [一句话简介]

## 仓库结构

src/server/  # 后端
src/web/     # 前端

## 技术栈

（/strategy 阶段确定后更新）

## 工作方式

新功能开发：/build → Explore → Spec → Demo → Build
Bug 修复：/fix
战略调整：/strategy
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
    "name": "[项目名称]",
    "description": "[简介]"
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

**下一步：** /strategy（制定产品战略）或 /build（开始版本交付）
```

## 强制规则

- 只新增文件和目录，不删除、不修改任何现有文件
- CLAUDE.md 必须备份原文件为 `.bak` 后才精简
- 每次只问一个问题
- 依赖缺失不阻塞，给出安装命令即可
