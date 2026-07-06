# 插件安装参考

> `/ds:init` 依赖检查时引用此文档。按此文档的安装命令执行即可。

## 使用方式

**所有依赖默认安装到项目级**，确保不同项目之间的依赖版本隔离，避免全局污染。

## 插件列表

### DreamSpec（本插件）

- **类型：** Claude Code 插件
- **用途：** AI Agent 创业开发团队协作 — 提供 `/ds:init` `/ds:vision` `/ds:build` `/ds:fix` 等命令
- **开源地址：** https://github.com/xingchen-xinyu/dreamspec
- **安装命令：**
  ```bash
  # Step 1: 注册 marketplace
  claude plugin marketplace add xingchen-xinyu/dreamspec --scope project
  # Step 2: 安装插件
  claude plugin install dreamspec@dreamspec-market --scope project
  ```
- **升级命令：**
  ```bash
  claude plugin marketplace update dreamspec-market
  claude plugin update dreamspec@dreamspec-market --scope project
  ```

### openspec

- **类型：** npm 全局包
- **用途：** Spec 编写 + Tasks 管理（explore → propose → apply）
- **开源地址：** https://github.com/Fission-AI/openspec
- **前置要求：** Node.js >= 20.19.0
- **安装命令：**
  ```bash
  npm install -g @fission-ai/openspec@latest
  cd <your-project>
  openspec init
  openspec config profile      # 选择 expanded workflows
  openspec update              # 应用到当前项目
  ```
- **升级命令：**
  ```bash
  npm install -g @fission-ai/openspec@latest
  cd <your-project>
  openspec update
  ```

### superpowers

- **类型：** Claude Code 插件
- **用途：** TDD + 多Agent + CodeReview + Debugging
- **开源地址：** https://github.com/obra/superpowers
- **安装命令：**
  ```bash
  claude plugins install superpowers@claude-plugins-official --scope project
  ```
- **升级命令：**
  ```bash
  claude plugin update superpowers@claude-plugins-official --scope project
  ```

### ui_ux_max_pro

- **类型：** Claude Code 插件 / npm 包（二选一）
- **用途：** HTML 原型设计
- **开源地址：** https://github.com/nextlevelbuilder/ui-ux-pro-max-skill
- **安装命令（推荐 Claude Code 插件方式）：**
  ```bash
  claude plugins marketplace add nextlevelbuilder/ui-ux-pro-max-skill
  claude plugins install ui-ux-pro-max@ui-ux-pro-max-skill --scope project
  ```
- **备选（npm 方式，Python 3.x 前置）：**
  ```bash
  npm install -g ui-ux-pro-max-cli
  cd <your-project>
  uipro init --ai claude
  ```
- **升级命令：**
  ```bash
  claude plugin marketplace update ui-ux-pro-max-skill
  claude plugin update ui-ux-pro-max@ui-ux-pro-max-skill --scope project
  ```

### frontend-design

- **类型：** Claude Code 插件
- **用途：** 前端 UI 组件设计 — 生成高质量、可维护的前端组件代码，支持设计系统驱动的 UI 开发
- **开源地址：** https://github.com/anthropics/claude-plugins-official（与 superpowers 同源，来自 claude-plugins-official marketplace）
- **安装命令：**
  ```bash
  claude plugins install frontend-design@claude-plugins-official --scope project
  ```
- **升级命令：**
  ```bash
  claude plugin update frontend-design@claude-plugins-official --scope project
  ```
