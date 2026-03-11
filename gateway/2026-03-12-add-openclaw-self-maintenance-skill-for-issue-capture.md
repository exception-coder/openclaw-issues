# 2026-03-12 · 新增 `openclaw-self-maintenance` skill，用于固化自身问题处理后的 issues 记录与推送

## 元信息

- **分类**：gateway
- **状态**：已彻底修复
- **影响范围**：当前机器 / 自身问题处理流程
- **相关组件**：skills / AGENTS / docs/issues / git
- **相关问题**：
  - [`../patterns/symptom-to-checks-to-fixes.md`](../patterns/symptom-to-checks-to-fixes.md)

## 问题现象

用户希望把以下行为固化成稳定流程：

- 当处理 OpenClaw 自身问题时自动记录 issues
- 更新索引与通用排查模式
- 记录完成后推送到 GitHub

并追问：

- 最佳方案是 skill 还是定时任务
- 自定义 skill 应该放在哪里
- 是否有标准约定能让系统自动读取

## 症状 → 优先排查项 → 修复命令

| 症状 | 优先排查项 | 修复命令/动作 |
|---|---|---|
| 自身问题处理流程靠手工记忆 | 是否需要一个固定工作流 | 编写专用 skill |
| 希望“修完就记录并推送” | 是否属于事件驱动而非时间驱动 | 主方案用 skill，定时任务只做兜底 |
| 想知道 skill 放哪里 | 是否存在本地自定义 skill 目录 | 放到 `~/.openclaw/skills/public/<skill-name>/` |

## 初步判断

这类需求本质上不是“定时整理”，而是“事件触发后的结构化收尾流程”。

因此更适合：

- 用 **skill** 固化主流程
- 用 `AGENTS.md` 加一条短规则做兜底
- 如有需要，再用定时任务做未推送检查

## 检查过程

### 1) 检查现有 skill 形态

确认 OpenClaw 自带 skills 都以 `SKILL.md` 形式存在，并遵循标准 skill 目录结构。

### 2) 检查本地自定义 skill 目录

当前 `~/.openclaw` 下还没有现成的本地自定义 skills，因此直接按推荐路径创建。

## 根因结论

> 需要一个本地自定义 skill 来固化“自身问题处理 → issues 记录 → 索引更新 → git push”的工作流，而不是依赖临时记忆或单纯定时任务。

## 采取的修复

### 最终修复

新增本地自定义 skill：

- `~/.openclaw/skills/public/openclaw-self-maintenance/SKILL.md`
- `~/.openclaw/skills/public/openclaw-self-maintenance/references/classification.md`

并在 workspace `AGENTS.md` 中补充兜底规则：

- 处理 OpenClaw 自身问题时优先使用 `openclaw-self-maintenance` skill
- 修复后记录 `docs/issues`
- 推送 `docs/issues` 仓库

## 验证结果

- skill 文件已落地
- AGENTS 兜底规则已补充
- 后续可作为标准工作流复用

## 风险与回滚

- 风险：若本地自定义 skill 目录未被当前运行环境自动扫描，则可能更多依赖 AGENTS 兜底规则
- 回滚方式：删除 skill 目录并移除 AGENTS 中对应规则

## 后续建议

1. 后续用真实案例继续迭代该 skill
2. 如需更强自动化，可增加“未推送 issues 变更检查”的 cron 作为兜底
3. 可后续把该 skill 打包成可分发 `.skill`

## 可复用经验

### 下次先查什么

1. 这是事件驱动流程还是时间驱动流程
2. 是否应优先做成 skill 而不是 cron
3. 是否需要在 AGENTS 中加兜底规则

### 关键词

- custom skill
- issues knowledge base
- self maintenance
- AGENTS fallback
- event driven workflow
