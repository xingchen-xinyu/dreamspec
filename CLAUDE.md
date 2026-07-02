# CLAUDE.md

## 项目

**DreamSpec Plugin** — Claude Code 插件，提供 AI Agent 创业开发团队协作能力。

## 仓库结构

```
skills/          # Skill 定义（编排器 + 角色）
  init.md        # /init 项目初始化
  strategy.md    # /strategy 战略规划
  build.md       # /build 版本交付
  fix.md         # /fix 问题修复
  ceo.md         # CEO 角色
  po.md          # PO 角色
  tl.md          # TL 角色
  qa.md          # QA 角色
templates/       # 项目模板
  state.json     # plugin-state.json 模板
```

## 技术说明

- 插件类型：Claude Code Plugin
- Skill 格式：Markdown with YAML frontmatter
- 依赖：openspec、superpowers、ui_ux_max_pro

## Skill 文件规范

- 编排器 Skill 负责流程调度和角色加载
- 角色 Skill 负责专业能力定义和规范约束
- 角色 Skill 只被编排器加载，不独立作为命令入口
- 强制规则使用"红线"标记，语气明确不容模糊

## 开发准则

- 修改 Skill 文件后无需重新安装，Claude Code 自动加载
- 测试方式：在新项目目录中安装插件，验证各命令行为
- 版本号遵循 semver
