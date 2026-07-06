# CLAUDE.md

## 项目

**DreamSpec Plugin** — Claude Code 插件，提供 AI Agent 创业开发团队协作能力。

## 仓库结构

```
skills/          # 命令入口（编排器）
  init/SKILL.md        # /ds:init 项目初始化/迁移/升级
  vision/SKILL.md      # /ds:vision 产品愿景
  build/SKILL.md       # /ds:build 版本交付
  fix/SKILL.md         # /ds:fix 问题修复
roles/           # 角色定义（不注册命令，被编排器加载）
  ceo.md               # CEO 角色
  po.md                # PO 角色
  tl.md                # TL 角色
  qa.md                # QA 角色
templates/       # 项目模板
  state.json     # plugin-state.json 模板
```

## 技术说明

- 插件类型：Claude Code Plugin
- Skill 格式：Markdown with YAML frontmatter
- 依赖：openspec、superpowers、ui_ux_max_pro、frontend-design

## Skill 文件规范

- 编排器 Skill 负责流程调度和角色加载
- 角色 Skill 负责专业能力定义和规范约束
- 角色定义放在 roles/ 目录，不注册命令入口，仅被编排器按需加载
- 强制规则使用"红线"标记，语气明确不容模糊

## 开发准则

- 修改 Skill 文件后无需重新安装，Claude Code 自动加载
- 测试方式：在新项目目录中安装插件，验证各命令行为
- 版本号遵循 semver
