# 2026-03-12 · 新增 `docs/capabilities` 目录，用于记录能力与实际案例

## 元信息

- **分类**：gateway
- **状态**：已彻底修复
- **影响范围**：当前 workspace / 文档体系
- **相关组件**：docs / capabilities / issues / workflow
- **相关问题**：
  - [`./2026-03-12-add-openclaw-self-maintenance-skill-for-issue-capture.md`](./2026-03-12-add-openclaw-self-maintenance-skill-for-issue-capture.md)

## 问题现象

用户希望新增一个独立目录，用于记录：

- 我的能力
- 有趣且真实完成过的工作
- 某项能力已经在真实场景中被验证的案例

并给出本次场景作为例子：

- 利用我和 skill 完成自动记录 issues 并自动推送到 GitHub

## 症状 → 优先排查项 → 修复命令

| 症状 | 优先排查项 | 修复命令/动作 |
|---|---|---|
| 只有 issues，没有能力档案 | 是否需要和“故障记录”分层 | 新建 `docs/capabilities/` |
| 想记录“能做什么”和“真实做过什么” | 是否区分稳定能力与案例 | 拆分 `abilities/` 与 `scenarios/` |
| 想把本次工作沉淀成可展示能力 | 是否已有代表性真实案例 | 将本次 skill + issues + push 过程写成 scenario |

## 初步判断

`docs/issues/` 更适合记录：

- 故障
- 排查
- 修复
- 复盘

但不适合单独承担：

- 能力档案
- 代表性工作成果
- “我已经稳定具备哪些能力”的展示性沉淀

因此应新增独立目录。

## 根因结论

> 需要把“问题知识库”和“能力/案例档案”分层管理，因此新增 `docs/capabilities/` 目录更合理。

## 采取的修复

### 最终修复

新增：

- `docs/capabilities/README.md`
- `docs/capabilities/INDEX.md`
- `docs/capabilities/abilities/auto-record-and-push-openclaw-issues.md`
- `docs/capabilities/scenarios/2026-03-12-openclaw-self-maintenance-workflow.md`

目录结构采用：

- `abilities/`：记录稳定能力
- `scenarios/`：记录真实案例

## 验证结果

- 目录已创建
- 已写入首批能力与案例文档
- 本次场景已作为代表性案例沉淀

## 后续建议

1. 后续每形成一个稳定能力，就补一条 `abilities/`
2. 每完成一个有代表性的真实工作，就补一条 `scenarios/`
3. 如有需要，后续可再增加 `playbooks/` 用于记录成体系工作方法

## 可复用经验

### 下次先查什么

1. 这是“故障修复”还是“能力沉淀”
2. 是应该写进 `issues/`，还是写进 `capabilities/`
3. 是否需要“能力 + 案例”双记录

### 关键词

- capabilities
- abilities
- scenarios
- knowledge base layering
- reusable work proof
