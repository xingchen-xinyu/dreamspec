---
name: ds:commit
description: 用户提交代码、commit、保存工作进度时使用
---

# /ds:commit — 提交记录

## 触发

- 显式调用：`/ds:commit`
- 自动触发：用户表达提交意图（"提交代码"、"commit"、"保存进度"、"这个做完了提交"等）

---

## 流程

### Step 1: 前置准备

1. 检查 `solution/worklog.json` 是否存在，不存在则创建：
   ```json
   { "unreleased": [] }
   ```
2. 读取 `plugin-state.json` 获取 `versionGoal`

### Step 2: 回顾改动

1. 查看代码改动（git diff / git status）
2. 按以下维度识别：

| 类型 | 判定依据 |
|------|---------|
| feature | 新增功能、新增页面、新增接口 |
| fix | Bug 修复、问题处理 |
| optimization | 性能优化、体验改进 |
| refactor | 代码重构、结构调整 |

3. 展示总结，让用户确认：

> 本次改动：
> - [feature] 用户管理 CRUD — 新增 UserController + UserService
> - [fix] 移动端布局修复 — 修复导航栏错位
>
> 确认提交？

### Step 3: 红线检查

逐项快速检查：

| 检查项 | 标准 |
|--------|------|
| 测试通过 | `npm test` 全量通过（或等效命令） |
| TDD 纪律 | 本次改动有测试覆盖 |
| 架构合规 | 分层纪律无违反（Controller/Service/Model 职责正确） |
| 代码整洁 | 无浮点 Promise、无 N+1、无魔法数字 |
| 安全基线 | 外部输入有校验、参数化查询、无敏感信息泄露 |
| 软删除 | 业务数据无物理删除 |

**微小调整判定**：若改动 < 30 行且纯样式/文案/配置变更，跳过红线检查直接提交。

检查结果：
- **全部通过** → 继续
- **有阻塞项**（测试未通过/架构违规/安全问题）→ 提示用户修复后再提交
- **有严重项** → 提示用户确认是否忽略

### Step 4: 生成 commit message

按规范格式生成，展示给用户确认（用户可手动修改）：

```
feat(模块): 用户管理 CRUD — 完成增删改查、列表分页筛选

- 新增 UserController、UserService
- 新增用户列表分页查询
- 新增用户创建/编辑/删除接口

Co-Authored-By: Claude <noreply@anthropic.com>
```

多类型混合时自动合并为一条 commit，分类型列出。

### Step 5: 执行 + 记录

1. 执行 `git add` + `git commit -m "..."` 
2. 追加记录到 `solution/worklog.json`：

```json
{
  "id": "wl-YYYYMMDD-NNN",
  "type": "feature|fix|optimization|refactor",
  "title": "简短标题",
  "summary": "实际完成的内容摘要",
  "scope": "versionGoal 或 null",
  "commits": ["<commit SHA>"],
  "createdAt": "<ISO timestamp>",
  "completedAt": "<ISO timestamp>"
}
```

ID 生成：`wl-YYYYMMDD-NNN`，NNN 为当日序号（当日已有记录数 +1）。

### Step 6: 汇报

> ✅ 已提交：`feat(模块): 用户管理 CRUD`
> commit: abc123
> worklog: wl-20260710-001
