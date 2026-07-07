---
name: ds:upgrade
description: 依赖管理 + 插件升级 — 检测与安装缺失依赖 → 更新 marketplace → 批量升级
---

# /ds:upgrade — 依赖管理 + 插件版本检测与升级

## 定位

`/ds:upgrade` 是 DreamSpec 的依赖管家——管安装、管升级、管合规。可在任意安装了 DreamSpec 插件的目录中独立运行，不依赖项目是否已初始化。

> **提示：** 如果直接运行 `/ds:init`，init 也会自动安装依赖（采用阻断策略）。无论先跑哪个命令，最终依赖都会就绪。

## 入口

用户输入 `/ds:upgrade` 触发。

## 前置条件

1. 执行 `claude plugins list` 检查输出中是否包含 `dreamspec`
   - 如果不存在 → 提示用户先安装 DreamSpec 插件（安装命令详见 [PLUGINS.md](../../reference/PLUGINS.md) DreamSpec 条目），流程终止
   - 如果存在 → 从输出中提取版本号，继续流程

2. 读取 [PLUGINS.md](../../reference/PLUGINS.md) 获取各插件的 marketplace 和升级命令

3. 检查 `.claude/plugin-state.json` 是否存在：
   - **存在** → 完整模式：安装 + 版本对比 + 升级 + 合规复检
   - **不存在** → 依赖模式：安装 + 版本检测（项目尚未初始化，跳过合规复检）

## 流程

### Step 1: 依赖检测与安装 ⚠️非阻断

> 自动检测并安装缺失的依赖插件。安装失败不阻断流程，记录警告后继续。

依赖检测采用**两级模式**：先检查设备级（全局是否安装），再检查项目级（当前项目是否已配置）。两级都满足才算"已就绪"。

**A. Claude Code 插件**（检查项目级是否就绪）：

执行 `claude plugins list` 获取已安装列表，然后检查项目的 `.claude/` 目录下是否有对应插件的配置目录：

| 依赖 | 用途 | 设备级检测 | 项目级检测 |
|------|------|-----------|-----------|
| superpowers | TDD + 多Agent + CodeReview + Debugging | `claude plugins list` 中是否包含 `superpowers` | `.claude/` 下是否有 superpowers 相关配置 |
| ui_ux_max_pro | HTML 原型设计 | `claude plugins list` 中是否包含 `ui-ux-pro-max` | `.claude/` 下是否有 ui-ux-pro-max 相关配置 |
| frontend-design | 前端 UI 组件设计与生成 | `claude plugins list` 中是否包含 `frontend-design` | `.claude/` 下是否有 frontend-design 相关配置 |

> **判定规则：** 设备级满足 + 项目级满足 → 已就绪，跳过安装。设备级满足但项目级缺失（如新项目、全局安装了但项目级没有）→ 仍需执行项目级安装。设备级缺失 → 完整安装。

**B. openspec**（检查项目级是否就绪）：

| 依赖 | 用途 | 设备级检测 | 项目级检测 |
|------|------|-----------|-----------|
| openspec | Spec 编写 + Tasks 管理 | `openspec --version` 是否可用 | 项目根目录下 `openspec/` 目录是否存在 |

> **判定规则：** 设备级（全局 npm 包）+ 项目级（`openspec/` 目录）两级都满足才算"已就绪"。**openspec 全局安装 ≠ 当前项目已配置**——每个项目必须独立执行 `openspec init` 生成 `openspec/` 目录。新项目目录下即使全局已安装 openspec，也必须执行项目级配置。

**C. 按需安装（只执行缺失的层级）：**

| 依赖 | 缺失情况 | 需执行的步骤 |
|------|---------|-------------|
| **openspec** | 仅项目级缺失（全局已安装） | ② `cd <项目根目录>` ③ `openspec init` ④ `openspec config profile`（选择 expanded workflows 扩展模式） ⑤ `openspec update` |
| **openspec** | 设备级 + 项目级都缺失 | ① `npm install -g @fission-ai/openspec@latest` ② `cd <项目根目录>` ③ `openspec init` ④ `openspec config profile` ⑤ `openspec update` |
| **superpowers** / **frontend-design** | 任意级别缺失 | `claude plugins install <plugin>@claude-plugins-official --scope project`（幂等，已安装则跳过） |
| **ui_ux_max_pro** | 任意级别缺失 | ① `claude plugins marketplace add nextlevelbuilder/ui-ux-pro-max-skill` ② `claude plugins install ui-ux-pro-max@ui-ux-pro-max-skill --scope project`（幂等） |

> **红线：** openspec 的项目级配置（步骤 ②③④⑤）是**最容易遗漏**的环节——设备全局已安装 openspec 时，AI 容易错误地判定为"已就绪"而跳过。必须检查 `openspec/` 目录是否存在，不存在就强制执行项目级配置。openspec config profile 和 update 是启用扩展模式（explore → propose → apply）的必要步骤，缺一不可。

**D. 安装后验证：**

每个依赖安装完成后，立即验证是否真正可用：
- Claude Code 插件：执行 `claude plugins list` 确认插件出现在列表中；
- openspec：执行 `openspec --version` 确认命令可用；**并且**检查项目目录下 `openspec/` 目录是否已生成

**E. 安装结果：**
- ✅ 全部安装成功 → 进入 Step 2
- ⚠️ 部分/全部安装失败 → **记录警告**，展示失败原因和手动安装命令，继续 Step 2（不阻断）

> 与 `/ds:init` Step 2 的区别：init 中依赖是后续流程的前置条件，安装失败必须终止（🚫阻断）；upgrade 中依赖安装失败不影响版本检测和升级流程（⚠️警告）。

### Step 2: 更新 marketplace + 获取 dreamspec 远端版本

1. 刷新所有相关 marketplace（仅刷新缓存，不修改插件文件）：

```
claude plugin marketplace update dreamspec-market
claude plugin marketplace update claude-plugins-official
claude plugin marketplace update ui-ux-pro-max-skill
```

任一 marketplace 更新失败 → 不阻塞，记录并继续。

2. 获取 dreamspec 远端最新版本（使用 Fetch 工具，不可读本地缓存）：

```
Fetch: https://raw.githubusercontent.com/xingchen-xinyu/dreamspec/main/.claude-plugin/plugin.json
```

从返回 JSON 的 `version` 字段获取远端最新版本号。

### Step 3: 汇总版本

执行 `claude plugins list` 获取已安装版本，结合 `openspec --version`，与 dreamspec 远端版本对比展示：

```markdown
📦 插件版本检测

| 插件 | 当前版本 | 最新版本 | 状态 |
|------|---------|---------|------|
| dreamspec | 1.3.1 | 1.3.2 | 🔄 可升级 |
| superpowers | 2.0.0 | — | ✅ 已是最新 |
| frontend-design | 1.2.0 | — | ✅ 已是最新 |
| ui_ux_max_pro | — | — | ⚠️ 未安装 |
| openspec | 1.5.0 | — | ✅ 已是最新 |
```

> 依赖插件不手动比对版本号——`claude plugin update` 命令自带"已是最新则跳过"能力，在 Step 5 执行时自动判断。

### Step 4: 用户确认

直接询问用户：

> 已更新所有 marketplace。是否尝试升级全部插件？
>
> 升级命令会自动跳过已是最新版本的插件，不会重复安装。

- 用户确认 → 进入 Step 5
- 用户跳过 → 流程结束

### Step 5: 执行升级

> **原理：** 不手动比对版本号（容易出错）。直接执行 `claude plugin update`，该命令本身就具备"已是最新则跳过"的能力。

按顺序执行升级，每个命令的输出即表明结果（已更新 / 已是最新 / 失败）：

**依赖插件：**
1. openspec：`npm install -g @fission-ai/openspec@latest` → `cd <项目根目录>` → `openspec config profile`（如尚未选择扩展模式） → `openspec update`
2. superpowers：`claude plugin update superpowers@claude-plugins-official --scope project`
3. frontend-design：`claude plugin update frontend-design@claude-plugins-official --scope project`
4. ui_ux_max_pro：`claude plugin marketplace update ui-ux-pro-max-skill` → `claude plugin update ui-ux-pro-max@ui-ux-pro-max-skill --scope project`

> Step 1 中已安装的依赖：`claude plugin update` 会自动判断为"已是最新"，无需特殊处理。
> Step 1 中安装失败的依赖：跳过升级步骤，报告中标注。

**主插件（最后执行）：**
5. dreamspec：`claude plugin update dreamspec@dreamspec-market --scope project`

**dreamspec 自升级特殊处理：**

如果 dreamspec 升级成功且用户输入包含 `--force` 关键词 → 强制升级模式，否则为增量更新：

- 增量更新（默认）：升级后检查是否需要补全目录、对比 CLAUDE.md 模板、迁移 plugin-state.json
- 强制升级（--force）：用远端最新版本覆盖本地 Skill 文件，保留用户数据

**升级失败处理：**
- 任一插件升级失败 → 不阻塞流程，记录失败信息，继续下一个
- 最终报告中标注失败项及手动修复命令

### Step 6: 合规复检

> **仅在项目已初始化时执行**（`.claude/plugin-state.json` 存在）。项目未初始化则跳过此步骤。

dreamspec 自身升级后，检查项目状态是否需要同步：

```
📋 合规复检

检查项：
  - 新增目录是否已补全（对比 plugin-state.json 中的 directories 与实际目录）
  - CLAUDE.md 是否包含所有必要章节（项目 / 仓库结构 / 技术栈 / 工作方式）
  - plugin-state.json 字段是否与最新模板一致
  - 如有不合规 → 提示用户运行 /ds:init 同步
```

### Step 7: 汇报结果

```markdown
## /ds:upgrade 完成

**📦 依赖安装：**
✅ superpowers（已安装，vX.X.X）
✅ frontend-design（新安装，vX.X.X）
✅ ui_ux_max_pro（已安装，vX.X.X）
⚠️ openspec 安装失败 — 手动执行：npm install -g @fission-ai/openspec@latest

**🔄 升级结果：**
| 插件 | 结果 |
|------|------|
| dreamspec | ✅ 已升级 v1.3.1 → v1.3.2 |
| superpowers | ✅ 已是最新 (vX.X.X) |
| frontend-design | ✅ 已是最新 (vX.X.X) |
| ui_ux_max_pro | ⚠️ 未安装，已跳过 |
| openspec | ✅ 已是最新 (vX.X.X) |

（如有失败项，列出原因和手动修复命令）

**📋 合规复检：**（仅已 init 项目展示）
✅ 项目结构完整，无需同步
（或：⚠️ 发现 X 项变更，建议运行 /ds:init 同步）
```

## 强制规则

- Step 1 依赖安装为非阻断步骤：安装失败记录警告，不影响后续版本检测和升级流程
- 不手动比对版本号，依赖 `claude plugin update` 命令自身的"已是最新则跳过"能力
- 升级前先更新所有 marketplace，确保获取最新版本信息
- 升级前必须用户确认，不自动升级
- 升级失败不阻断流程，记录失败信息，继续下一个插件
- 只升级插件本身，不修改用户源代码文件（src/ 等）
- Step 6 合规复检仅在项目已初始化时执行
