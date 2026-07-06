---
name: ds:vision
description: 产品愿景 — CEO 主导，PO/TL 参与，制定产品愿景和版本路线图
---

# /ds:vision — 产品愿景

## 入口

用户输入 `/ds:vision` 触发。

## 流程

### 阶段 1: 准备

1. 检查 `solution/vision.md` 是否存在
   - 已存在 → 提示用户：本次是更新愿景还是重新制定？
   - 不存在 → 进入新愿景制定流程
2. 确保 CEO/PO/TL 角色 Skill 可用

### 阶段 2: CEO 主导 — 商业分析

**加载 CEO 角色 Skill（roles/ceo.md）中的角色和能力。**

CEO 按以下步骤推进：

1. 理解用户想法，一次只问一个问题澄清
2. 从商业价值评估，不同意时提出反对理由
3. 确定产品定位（一句话 + 目标用户 + 差异化）
4. 用精益画布梳理商业模型
5. 分析竞品格局
6. 规划版本路线图

### 阶段 3: PO + TL 参与 — 可行性评估

CEO 形成草案后，邀请 PO 和 TL 参与：

- **PO（加载 roles/po.md）：** 从产品角度评估——需求是否完整？用户场景是否合理？版本划分是否可行？
- **TL（加载 roles/tl.md）：** 从技术角度评估——技术可行性？关键风险？资源需求？

### 阶段 4: CEO 整合

CEO 综合 PO 和 TL 的反馈，形成最终愿景方案。

### 阶段 5: 目录扩展

根据确认的技术选型，在 `src/` 下新增对应目录（如 wechat、timer），只新增不删除。
更新 `plugin-state.json` 中的 `techStack` 和 `directories`。

### 阶段 6: 用户确认

完整呈现愿景方案：
1. 产品定位与商业模型
2. 版本路线图
3. 技术栈确认
4. 目录结构

等待用户确认后写入 `solution/vision.md`，更新 `plugin-state.json` 的 `vision_completed = true`。

### 阶段 7: 项目信息回填

愿景方案确认后，将产出信息回填到项目元数据中，完成 init 阶段留下的草稿信息：

1. 从 `solution/vision.md` 提取：
   - **产品名称**：从「产品定位」章节提取（如 vision 阶段确定了更精确的名称，则覆盖 init 阶段的目录名；否则保留目录名）
   - **一句话描述**：从「产品定位」章节提取一句定位语（覆盖 init 阶段的空字符串）
   - **技术栈**：从确认的技术选型中提取完整列表

2. 更新 `.claude/plugin-state.json`：
   - `project.name` → 最终产品名称
   - `project.description` → 一句话描述
   - `phases.vision_completed` → `true`
   - `techStack` → 完整技术栈信息（已在阶段 5 更新）

3. 更新 `CLAUDE.md` 中的占位内容：
   - 将 `{{PROJECT_INFO}}` 替换为实际产品名称和一句话描述（如 `产品名称 — 一句话描述`）
   - 将 `{{TECH_STACK}}` 替换为实际技术栈列表
   - 保留用户已在 CLAUDE.md 中手动添加的其他内容，只做确定性替换

4. 汇报回填结果：
   ```markdown
   **项目信息已回填：**
   - 产品名称：[name]
   - 一句话描述：[description]
   - 技术栈：[techStack 摘要]
   - 已更新：CLAUDE.md、plugin-state.json
   ```

## 产出物

- `solution/vision.md` — 产品愿景文档
- 更新 `plugin-state.json` — 回填 project.name/description + techStack + directories + vision_completed
- 更新 `CLAUDE.md` — 回填项目名称、描述、技术栈

## 强制规则

- 每次只问一个问题，不要一次抛出多个问题
- 愿景方案必须得到用户确认后才能写入文件
- CEO 是战略决策者，PO/TL 是评估参与者，不替代 CEO 做战略判断
