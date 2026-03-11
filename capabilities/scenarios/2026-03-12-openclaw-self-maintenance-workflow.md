# 2026-03-12 · 利用 skill + knowledge 仓库实现“修复即归档、归档即推送”工作流

## 场景说明

本次实际完成了一个高价值的工作流建设：

> 不只是修复 OpenClaw 问题，而是把“问题处理后的知识沉淀”也做成标准动作。

目标是：

- 当处理 OpenClaw 自身问题时
- 自动记录到 `docs/openclaw-knowledge/issues`
- 更新索引与通用排查模式
- 在形成复用价值时补充到 `docs/openclaw-knowledge/capabilities`
- 并推送到 GitHub 仓库

## 这次实际做成了什么

### 1. 修复了一组真实问题

包括：

- Control UI 出现 `gateway token missing`
- Telegram 会话看似丢失 / 已读不回
- node host disconnected
- Node service `not loaded`
- `exec` 无法使用

### 2. 建立并重构知识库结构

完成了：

- `docs/openclaw-knowledge/` 父目录建立
- `docs/openclaw-knowledge/issues/` 用于故障、排查、修复、复盘
- `docs/openclaw-knowledge/capabilities/` 用于能力、方法、实战案例
- 父目录 `README.md` 与 `INDEX.md`
- issues 下分类目录、`INDEX.md`、`TEMPLATE.md`
- `patterns/symptom-to-checks-to-fixes.md`
- 多篇真实问题记录

### 3. 将 `docs/openclaw-knowledge` 作为独立知识仓库维护

这样可以：

- 不影响整个 workspace 的 git 结构
- 单独维护 OpenClaw 知识资产
- 同时纳入 `issues` 与 `capabilities`
- 让“问题记录”和“能力沉淀”在同一个仓库内闭环

### 4. 推送到 GitHub

已完成推送到：

- `https://github.com/exception-coder/openclaw-issues.git`

### 5. 把这套行为固化为 agent + skill 协同机制

这次实战里，真正起关键作用的是两层机制：

- **AGENTS.md 兜底规则**：规定什么时候应该进入这套流程
- **`openclaw-self-maintenance` skill**：规定进入流程后具体怎么做

这两个部分缺一不可。

---

## 实现这套工作流的两个核心关键

## 关键一：AGENTS 片段（负责触发与兜底）

位置：

- `/Users/zhangkai/.openclaw/workspace/AGENTS.md`

本次生效的关键规则片段是：

> When handling OpenClaw's own operational problems (gateway, Control UI, node service, sessions, channels, auth, connectivity, exec/system.run, recovery), prefer the `openclaw-self-maintenance` skill. After meaningful diagnosis or repair, record the case in `docs/openclaw-knowledge/issues`, update shared troubleshooting notes when useful, and when the work has formed a reusable method or validated operational capability, also record it in `docs/openclaw-knowledge/capabilities`. Push the `docs/openclaw-knowledge` knowledge repo after meaningful updates.

### 这个 AGENTS 片段的作用

它解决的是“**什么时候应该进入知识沉淀流程**”的问题：

1. **明确场景边界**
   - 只要是 OpenClaw 自身运维问题，就优先走专用 skill
   - 典型范围包括 gateway、Control UI、node service、sessions、channels、auth、connectivity、`exec/system.run`

2. **把记录动作从“可选”变成“默认动作”**
   - 不是修完就结束
   - 而是修完后默认要补 `issues`

3. **把“故障文档”升级成“能力文档”**
   - 如果这次工作已经形成了可复用方法，或者验证了一个稳定能力
   - 就进一步补到 `capabilities`

4. **把推送动作纳入闭环**
   - 有实质更新后，推送 `docs/openclaw-knowledge` 仓库

换句话说，AGENTS 片段负责定义：

- **何时触发**
- **至少要做到什么程度**
- **修复后不能漏掉哪些收尾动作**

---

## 关键二：Skill 片段（负责执行流程）

位置：

- `~/.openclaw/skills/public/openclaw-self-maintenance/SKILL.md`

Skill 的核心定义包括两部分。

### 2.1 Frontmatter：定义 skill 的使用范围

```md
---
name: openclaw-self-maintenance
description: Record and maintain an issues knowledge base when handling OpenClaw's own problems, including gateway, control UI, node service, sessions, channels, auth, connectivity, execution, and recovery issues. Use when diagnosing or fixing OpenClaw itself; when changing configuration to restore service; when the user asks to record, archive, or push problem handling notes; or when a reusable troubleshooting pattern emerges from a repair.
---
```

这部分的作用是让 agent 在遇到这些场景时，知道应该加载这个 skill。

### 2.2 Skill 主体：定义进入后怎么做

Skill 当前的核心流程可以概括为：

1. **先判断是不是 OpenClaw 自身问题**
   - gateway
   - Control UI
   - node / node service / LaunchAgent
   - sessions / channels
   - auth / token / proxy / pairing / recovery
   - exec / `system.run`

2. **如果修复或诊断有复用价值，就不要只停在修复**
   - skill 明确要求：修完 OpenClaw 自身问题后，不要止步于 fix
   - 还要补知识库记录

3. **先给问题分类**
   - `ui/`
   - `channels/`
   - `gateway/`
   - `nodes/`
   - `sessions/`
   - `patterns/`

4. **按照模板写 case**
   - 元信息
   - 问题现象
   - 症状 → 优先排查项 → 修复命令
   - 初步判断
   - 检查过程
   - 根因结论
   - 采取的修复
   - 验证结果
   - 风险与回滚
   - 后续建议
   - 可复用经验

5. **更新共享知识**
   - 更新 `INDEX.md`
   - 必要时更新 `patterns/symptom-to-checks-to-fixes.md`
   - 目录结构变化时更新 `README.md`

6. **提交并推送仓库**
   - `git status --short`
   - `git add .`
   - `git commit -m "<clear summary>"`
   - `git push`

7. **向用户汇报**
   - 修了什么
   - 记录了什么
   - 是否已推送
   - 还有什么风险或后续项

### 这个 Skill 的核心价值

Skill 解决的是“**进入流程后如何稳定执行**”的问题：

- 把问题从聊天内容变成结构化 case
- 把零散排障动作变成可审计的知识资产
- 把问题记录和仓库推送做成固定动作
- 降低未来重复排查的成本

---

## agent 与 skill 的协同关系

这次案例真正可复用，不是因为有单个文档，而是因为形成了下面这个协同模型：

### AGENTS.md 负责：

- 规定在什么场景下优先启用这个机制
- 做全局兜底，防止修完不记录
- 规定修复后要写 issues，必要时补 capabilities，并推送知识仓库

### SKILL.md 负责：

- 规定进入机制之后的具体步骤
- 给出分类、模板、索引、patterns、git 提交推送这些标准动作
- 约束记录质量，避免只写结论、不写排查过程

### 两者合起来的效果

- AGENTS 解决“要不要做”
- Skill 解决“具体怎么做”

这也是这次工作流能真正落地的核心。

---

## 为什么这个场景有代表性

因为它体现的不是单点修复，而是组合能力：

- 排障能力
- 配置修复能力
- 文档整理能力
- 知识库抽象能力
- git 仓库边界控制
- skill 驱动流程设计能力
- agent 规则与 skill 协同能力

## 可复用价值

后续凡是出现 OpenClaw 自身问题，都可以复用这次形成的模式：

1. 先修复问题
2. 再结构化记录到 `issues`
3. 抽象共性经验到 `patterns`
4. 在形成稳定方法后补到 `capabilities`
5. 提交并推送知识仓库

## 后续可继续优化的点

当前 skill 里的路径说明仍偏向旧的 `docs/issues` 结构，后续最好同步更新为：

- `docs/openclaw-knowledge/issues`
- `docs/openclaw-knowledge/capabilities`
- `docs/openclaw-knowledge` 仓库根

这样可以让 **AGENTS 规则、目录结构、skill 文案** 三者完全一致。

## 关键词

- self maintenance
- issues knowledge base
- capabilities knowledge base
- auto record
- auto push
- skill driven workflow
- AGENTS + skill
- OpenClaw ops
