# CLAUDE.md

## 项目

**DreamSpec Plugin** — Claude Code 插件，提供 AI Agent 创业开发团队协作能力。

## 仓库结构

```
skills/          # 所有 Skill（命令 + 角色）
  init/SKILL.md        # /ds:init 项目初始化/迁移
  upgrade/SKILL.md     # /ds:upgrade 插件版本检测与升级
  vision/SKILL.md      # /ds:vision 产品定位
  ceo/SKILL.md         # CEO — 产品定位方法论（仅 vision 加载）
  plan/SKILL.md        # /ds:plan 版本规划
  po/SKILL.md          # PO — 产品方案（通用 + PRD/原型专项，自动触发）
  tl/SKILL.md          # TL — 技术交付（通用 + 后端/前端/原型专项，自动触发）
  commit/SKILL.md      # /ds:commit 提交记录（自包含）
  release/SKILL.md     # /ds:release 发版（自包含）
templates/       # 项目模板
  state.json     # plugin-state.json 模板
```

## 设计理念

- **工程要求随身带**：角色 = 随身工程规范，不指挥用户怎么干活，只确保工程底线不丢失
- **不侵入开发过程**：用户正常使用 openspec/superpowers/ux_ui 等任意工具，角色要求自动融入
- **按需加载**：意图识别（关键词+上下文）→ 自动匹配角色 Skill → 内部按工作子类激活专项要求
- **commit 和 release 是存档点**：日常开发零打扰，只在提交和发版时主动介入

## 技术说明

- 插件类型：Claude Code Plugin
- Skill 格式：Markdown with YAML frontmatter
- 角色通过 Skill 系统的 description 自动匹配触发，无需用户显式调用
- 命令 Skill 负责前置检查；角色 Skill 负责工程规范定义，内部按子类分层

## 开发准则

- 修改 Skill 文件后无需重新安装，Claude Code 自动加载
- 测试方式：在新项目目录中安装插件，验证各命令行为
- 版本号遵循 semver，每次修改自动识别变更性质并升级版本：
  - **Major**（X.0.0）：破坏性变更 — 移除命令、重命名入口、不兼容的配置格式变更
  - **Minor**（x.Y.0）：新增功能 — 新增命令入口、新增 Skill、新增角色
  - **Patch**（x.y.Z）：修复与优化 — Bug 修复、文档更新、流程优化、代码清理
  - 版本号必须同步更新 `.claude-plugin/plugin.json` 和 `.claude-plugin/marketplace.json` 两处，缺一不可
- 插件每次升级时，检查 README.md 是否需要同步更新（新增命令、流程变化、配置变更等），确保文档与最新版本一致
