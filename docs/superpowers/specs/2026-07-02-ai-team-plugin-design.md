# AI Team Plugin — 设计文档

## 一、定位

将 openspec、superpowers、ui_ux_max_pro 等技能的开发流程和角色协作方法固化为一个 Claude Code 插件。通过 CEO/PO/TL/QA 四个 AI 角色按标准流程协作，帮用户把想法从战略规划到编码交付完整落地。

插件做的是**流程编排和角色视角注入**，不重写下层能力。

## 二、命令入口

| 命令 | 用途 | 主导角色 | 频率 |
|------|------|---------|------|
| `/init` | 项目初始化/迁移 | 编排器 | 新项目一次，切换时一次 |
| `/strategy` | 战略规划（定位、路线图） | CEO 主导 + PO/TL 参与 | 0→1 必须，后续按需 |
| `/build` | 版本交付（Explore→Spec→Demo→Build） | 见流程 | 每个版本 |
| `/fix` | 问题修复（定位→修复→QA验证） | TL 主导 + QA | 按需 |

## 三、角色设计

| 角色 | 核心能力 | 强制规范 |
|------|---------|---------|
| **CEO** | 商业模式分析、产品定位、演进策略、ROI评估 | 精益画布、MoSCoW、MECE |
| **PO** | 需求拆解、Spec编写、HTML原型设计、版本规划 | 移动端优先、WCAG 2.1 AA、Tailwind + shadcn/ui |
| **TL** | 技术方案、TDD开发、整洁架构、问题定位修复 | TDD 强制、整洁架构 DDD、OWASP Top 10、UT覆盖率 ≥ 80% |
| **QA** | 全流程质量审查（方案/代码/安全/规范） | 分级问题清单（阻塞/严重/建议）、不自行修复 |

### 角色触发关系

```
/strategy:   CEO(主导) + PO + TL
/build:
  注入角色要求 → openspec(explore→propose→apply) → Demo → Build
  参与角色:       PO(主导) + CEO + TL + QA
/fix:        TL(主导) + QA
```

## 四、场景流程

### 4.1 /init — 项目初始化/迁移

```
1. 项目识别 → 全新 or 已有
2. 配置 → 项目名称、简介、技术栈偏好
3. 创建目录结构（只新增，不删除）
4. 依赖检查 → openspec / superpowers / ui_ux_max_pro
5. CLAUDE.md 处理 → 递归处理根目录及所有子目录
    全新：生成轻量版骨架
    已有：精简规范细节（原文件备份 .bak）
    子目录：同样精简，保留技术栈 + 命令参考
6. 写入 plugin-state.json
7. 版本检查 + 升级支持
```

**全新项目目录结构：**

```
项目根目录/
├── solution/
├── src/
│   ├── server/       # 默认
│   └── web/           # 默认
├── tests/
├── .claude/
│   └── plugin-state.json
└── README.md
```

**已有项目迁移原则：** 只新增文件和目录，不删除、不修改任何现有文件。CLAUDE.md 精简后原文件备份为 `.bak`。

### 4.2 /strategy — 战略规划

```
CEO 主导 → 商业价值分析、产品定位、目标市场
PO + TL 参与 → 产品可行性评估 + 技术可行性评估
CEO 整合 → 形成最终战略方案
→ 用户确认

产出: solution/strategy.md
```

**strategy.md 结构：** 产品定位、商业模型、竞品分析、版本路线图、关键假设与风险。

**按技术栈扩展目录：** /strategy 确认技术选型后，在 src/ 下新增对应目录（如 wechat、timer、admin-web），只新增不删除。

### 4.3 /build — 版本交付

`/build` 不自己 Explore，而是把角色要求注入 openspec，直接复用其 explore→propose→apply 链路。

```
编排器准备阶段：
  1. 从 plugin-state.json 读取当前版本号，自动推断下一版本
     · 默认递增末位（v1.0 → v1.1），大版本升级由用户在 Explore 时指定
  2. 读取 solution/strategy.md（CEO 战略视角）
  3. 组装角色要求上下文：

     注入到 openspec explore：
     · CEO 视角：本次版本与战略对齐？商业优先级？
     · PO 视角：需求范围？用户价值？版本目标？
     · TL 视角：技术可行性？风险？
     · QA 视角：可验证性？

Step 1: Explore + Spec（openspec，一次连贯完成）
  openspec explore → propose
  上下文已注入角色要求，openspec 内部自主探索后直接进入 propose
  propose 产出 = specification.md = PRD
  → 用户确认 spec（openspec 内部探索中间无暂停）

Step 2: Demo（ui_ux_max_pro）
  PO 主导 + QA
  产出: solution/v{x}/demo/ HTML 原型
  → 用户确认原型

Step 3: Build（openspec apply + superpowers）
  TL 主导 + QA
  openspec tasks → superpowers TDD → 多Agent并行 → CodeReview
  → 用户确认，版本交付完成
```

**本插件在 /build 中做的事：**
- explore 前注入带角色视角的上下文
- propose 后插入 Demo 环节
- Build 阶段注入 superpowers 能力

**版本号管理：**

plugin-state.json 记录 `currentVersion`，/build 触发时：
- 默认递增末位：v1.0 → v1.1 → v1.2
- 大版本升级（如 2.0）由用户在 Explore 确认时指定
- 首次 /build 版本从 v1.0 开始

**新版本 vs 迭代版差异：**

| 环节 | 新版本 | 迭代版 |
|------|--------|--------|
| Explore | 战略对齐，定义版本目标 | 范围锁定，影响评估 |
| Spec | 完整 specification.md | 变更 specification.md（diff 视角） |
| Demo | 完整原型 | 变更页面原型 |
| Build | 完整功能开发 | 增量修改 + 回归测试 |
| QA 重点 | 架构合规、测试覆盖 | 回归覆盖、兼容性 |

### 4.4 /fix — 问题修复

```
TL 定位（← superpowers systematic-debugging）
  → 问题分析报告
  → 用户确认修复方案

TL 修复（← superpowers TDD）
  → 先写复现测试 → 修复 → 回归

QA 验证（本插件 QA + superpowers verification-before-completion）
  → 验证报告
  → 通过则完成，未通过回到定位
```

## 五、能力映射

```
/strategy:  本插件（角色协作）
/build:
  注入角色上下文 → openspec(explore→propose→apply)
  Demo:      ui_ux_max_pro（插入在 propose 之后）
  Build:     openspec apply + superpowers(TDD/多Agent/CodeReview)
/fix:
  定位:      superpowers systematic-debugging
  修复:      superpowers TDD
  验证:      本插件 QA + superpowers verification-before-completion
/init:       本插件（编排 + 目录 + 依赖 + 升级，递归处理各级 CLAUDE.md）
```

## 六、插件项目结构

```
DreamSpec/                    # GitHub 仓库
├── plugin.json             # 插件清单 + 依赖声明
├── skills/
│   ├── init.md             # /init 编排器
│   ├── strategy.md         # /strategy 编排器
│   ├── build.md            # /build 编排器
│   ├── fix.md              # /fix 编排器
│   ├── ceo.md              # CEO 角色
│   ├── po.md               # PO 角色
│   ├── tl.md               # TL 角色
│   └── qa.md               # QA 角色
├── templates/
│   ├── scope.md            # scope 产出模板
│   └── state.json          # plugin-state.json 模板
├── README.md               # 使用文档
└── CLAUDE.md               # 插件自身的开发规范
```

### plugin.json

```json
{
  "name": "DreamSpec",
  "version": "1.0.0",
  "description": "AI Agent 创业开发团队 — CEO/PO/TL/QA 协作开发",
  "dependencies": {
    "openspec": "latest",
    "superpowers": "latest",
    "ui_ux_max_pro": "latest"
  }
}
```

## 七、安装方式

```bash
# 全新项目
mkdir my-app && cd my-app
claude plugins install github:用户名/DreamSpec
# 在 Claude Code 中: /init → /strategy → /build

# 已有项目
cd existing-project
claude plugins install github:用户名/DreamSpec
# 在 Claude Code 中: /init（无痛迁移）→ /build
```

一条命令完成插件 + 所有依赖安装。
