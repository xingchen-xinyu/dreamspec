---
name: strategy
description: 战略规划 — CEO 主导，PO/TL 参与，制定产品战略和版本路线图
---

# /strategy — 战略规划

## 入口

用户输入 `/strategy` 触发。

## 流程

### 阶段 1: 准备

1. 检查 `solution/strategy.md` 是否存在
   - 已存在 → 提示用户：本次是更新战略还是重新制定？
   - 不存在 → 进入新战略制定流程
2. 确保 CEO/PO/TL 角色 Skill 可用

### 阶段 2: CEO 主导 — 商业分析

**加载 CEO 角色 Skill（skills/ceo/SKILL.md）中的角色和能力。**

CEO 按以下步骤推进：

1. 理解用户想法，一次只问一个问题澄清
2. 从商业价值评估，不同意时提出反对理由
3. 确定产品定位（一句话 + 目标用户 + 差异化）
4. 用精益画布梳理商业模型
5. 分析竞品格局
6. 规划版本路线图

### 阶段 3: PO + TL 参与 — 可行性评估

CEO 形成草案后，邀请 PO 和 TL 参与：

- **PO（加载 skills/po/SKILL.md）：** 从产品角度评估——需求是否完整？用户场景是否合理？版本划分是否可行？
- **TL（加载 skills/tl/SKILL.md）：** 从技术角度评估——技术可行性？关键风险？资源需求？

### 阶段 4: CEO 整合

CEO 综合 PO 和 TL 的反馈，形成最终战略方案。

### 阶段 5: 目录扩展

根据确认的技术选型，在 `src/` 下新增对应目录（如 wechat、timer），只新增不删除。
更新 `plugin-state.json` 中的 `techStack` 和 `directories`。

### 阶段 6: 用户确认

完整呈现战略方案：
1. 产品定位与商业模型
2. 版本路线图
3. 技术栈确认
4. 目录结构

等待用户确认后写入 `solution/strategy.md`，更新 `plugin-state.json` 的 `strategy_completed = true`。

## 产出物

- `solution/strategy.md` — 战略规划文档
- 更新 `plugin-state.json` — 技术栈和目录信息

## 强制规则

- 每次只问一个问题，不要一次抛出多个问题
- 战略方案必须得到用户确认后才能写入文件
- CEO 是战略决策者，PO/TL 是评估参与者，不替代 CEO 做战略判断
