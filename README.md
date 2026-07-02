# AI Team — AI Agent 创业开发团队

将 openspec、superpowers、ui_ux_max_pro 等技能的开发流程和角色协作方法固化为一个 Claude Code 插件。通过 CEO/PO/TL/QA 四个 AI 角色按标准流程协作，帮你把想法从战略规划到编码交付完整落地。

## 安装

```bash
claude plugins install github:用户名/DreamSpec
```

安装后自动注册以下命令：`/init`  `/strategy`  `/build`  `/fix`

依赖自动安装：openspec + superpowers + ui_ux_max_pro

## 快速开始

### 全新项目

```bash
mkdir my-app && cd my-app
claude plugins install github:用户名/DreamSpec
```

在 Claude Code 中：

```
/init          # 配置项目信息，创建目录结构
/strategy      # 定位产品、规划版本路线
/build         # 按版本迭代交付（Explore → Spec → Demo → Build）
```

### 已有项目

```bash
cd existing-project
claude plugins install github:用户名/DreamSpec
```

在 Claude Code 中：

```
/init          # 检测项目状态，无痛迁移，不删除任何现有文件
/build         # 继续当前版本的开发
```

## 命令参考

| 命令 | 用途 | 何时用 |
|------|------|--------|
| `/init` | 项目初始化/迁移 | 新项目开始、切换到插件模式 |
| `/strategy` | 战略规划 | 0→1 必须，后续按需调整战略 |
| `/build` | 版本交付 | 每个版本迭代 |
| `/fix` | 问题修复 | Bug 修复 |

## 工作流

### /build 四步

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
| **CEO** | 商业价值、战略方向 | `/strategy`（主导）、`/build` Explore |
| **PO** | 产品设计、Spec、原型 | `/build` Spec + Demo（主导） |
| **TL** | 技术方案、编码交付 | `/build` Build（主导）、`/fix`（主导） |
| **QA** | 质量审查、规范检查 | 全流程贯穿 |

## 目录结构

```
项目根目录/
├── solution/          # 战略、Spec、原型
│   ├── strategy.md    # 战略规划
│   └── v{x}/
│       ├── specification.md  # PRD（openspec 输出）
│       ├── tech-design.md    # 技术方案
│       └── demo/             # HTML 原型
├── src/
│   ├── server/        # 后端
│   ├── web/           # 前端
│   └── ...            # /strategy 后按技术栈扩展
├── tests/
├── .claude/
│   └── plugin-state.json
└── README.md
```

## FAQ

**Q: 和直接使用 openspec/superpowers 有什么区别？**

A: 本插件是它们的上层编排——提供角色视角、阶段划分、协作流程。你不再需要自己记住"什么时候用哪个技能"。

**Q: 已有项目会不会被破坏？**

A: `/init` 只新增文件和目录，不删除、不修改任何现有文件。原 CLAUDE.md 会备份为 `.bak`。

**Q: 每个阶段必须走完吗？**

A: 每步完成后你可以选择跳过或调整。流程是骨架，你是决策者。

**Q: 无前端的后端项目 Demo 步骤怎么处理？**

A: `/build` Explore 阶段可以声明跳过 Demo，纯后端项目可省略。
