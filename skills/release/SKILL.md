---
name: ds:release
description: ds:release 命令 — 发版、生成 changelog 并归档
---

# /ds:release — 发版

## 触发

手动调用 `/ds:release`。

---

## 流程

### Step 1: 前置准备

1. 读取 `solution/worklog.json`，获取 unreleased 列表
2. 读取 `solution/version.json`，获取 `currentVersion`、`versionGoal`
3. 读取 `solution/vision.md`（如存在），了解产品战略

### Step 2: 汇总待发版内容

如果 unreleased 为空 → "没有待发版的内容。"

展示汇总：

```
📦 待发版内容（{N} 项）：

• [feature] 用户权限管理 — wl-20260709-001
• [fix] 移动端登录布局修复 — wl-20260708-001
• [optimization] 用户列表查询性能优化 — wl-20260707-001

当前版本：v{currentVersion}
```

### Step 3: 判定版本号

按 semver 自动判定：

| 条件 | 版本类型 |
|------|---------|
| `currentVersion == "0.0.0"`（首次）| 询问：v1.0.0 正式版 还是 v0.1.0 预览版 |
| 用户明确指定（"发 1.0"）| 以用户指定为准 |
| 含 feature/opt/refactor，0.x 系列 | Minor：v0.{y+1}.0 |
| 含 feature/opt/refactor，1.0+ | Minor：v{x}.{y+1}.0 |
| 全 fix | Patch：v{x}.{y}.{z+1} |

> 建议新版本：**v1.3.0** (Minor)
> 确认？或输入其他版本号：

### Step 4: 战略检查

如果 `versionGoal` 匹配本次发版（如 versionGoal=1.0.0，用户发 1.0）：

1. 对照 vision.md 检查目标达成
2. 输出战略对齐评估：

> 🎯 版本目标 v1.0.0 达成检查：
> - vision 路线图定位：MVP 闭环验证
> - 已完成核心：用户注册登录、基础数据管理、社区动态
> - 待完成：LLM+RAG 基础版（评估中）
>
> 建议：{达到发版标准 / 建议完成 XX 后再发}

用户确认是否发版。

### Step 5: 发版质量检查

| 检查项 | 标准 |
|--------|------|
| 全量测试 | `npm test` 通过 |
| 跨功能一致性 | 无文件/模块被多个工作项冲突修改 |
| 代码审查 | 主要功能已完成审查 |
| 回归验证 | 核心路径正常 |

如有问题 → 列出清单 → 用户决定处理方式。

### Step 6: 生成版本档案

1. 创建版本目录 `solution/v{major}/v{major}.{minor}.{patch}/`
2. 生成版本完整日志 `changelog.md`：

```markdown
# v{x.y.z} — {版本标题}
- 日期：{date}
- 类型：{major/minor/patch}
- 版本目标：{versionGoal 或 "日常迭代"}

## 新增功能
### {功能名}
- commit {sha} — {摘要}

## 修复
### {修复名}
- commit {sha} — {摘要}

## 优化
### {优化名}
- commit {sha} — {摘要}
```

3. 将 unreleased 记录移入版本目录 `worklog.json`
4. 追加根目录 `CHANGELOG.md`（每版一行，新版本追加在顶部）：

```markdown
## v1.1.0 (2026-08-15) — 用户管理模块
- [feature] 用户 CRUD、权限分配
- [fix] 移动端布局修复
```

5. 如有 `versionPlan`，更新 `solution/version-plan.md` 对应行的"实际交付"列

### Step 7: 收尾

1. 更新 `solution/version.json`：
   - `currentVersion` → 新版本号
   - `versionGoal` → null（发版版本匹配时清空）
   - `versions[]` 追加新版本记录
2. 清空 `solution/worklog.json` 的 unreleased 数组
3. 打 git tag：`v{x.y.z}`
4. 汇报：

> ✅ v{x.y.z} 已发布
> - 包含 {N} 个工作项
> - 版本档案：solution/v{major}/v{x.y.z}/
> - CHANGELOG.md 已更新
> - Git tag：v{x.y.z}
