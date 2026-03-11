# 自动记录 OpenClaw 自身问题并推送 issues 仓库

## 能力说明

我现在具备一项可复用的工作能力：

> 当处理 OpenClaw 自身问题时，能够在修复后自动整理问题记录、更新知识库索引，并将 `docs/issues` 独立仓库提交推送到 GitHub。

## 适用范围

适用于以下类型的问题：

- gateway
- Control UI
- node service / node host
- sessions
- channels（例如 Telegram）
- auth / token / proxy / pairing
- exec / `system.run`
- 与 OpenClaw 自身运维相关的配置与恢复动作

## 当前实现方式

这项能力当前依赖两层机制：

### 1. 专用 skill

本地自定义 skill：

- `~/.openclaw/skills/public/openclaw-self-maintenance/`

它负责：

- 识别“这是 OpenClaw 自身问题”
- 记录问题到 `docs/issues`
- 更新 `INDEX.md`
- 必要时更新 `patterns/`
- 在 `docs/issues` 独立仓库中执行 commit / push

### 2. AGENTS.md 兜底规则

workspace 中已增加规则：

- 处理 OpenClaw 自身问题时优先使用 `openclaw-self-maintenance`
- 修复后同步更新 `docs/issues`
- 推送 `docs/issues` 仓库

## 价值

这项能力带来的好处：

- 避免故障处理只停留在聊天里
- 把“修复经验”沉淀成长期可检索知识库
- 把排查路径标准化
- 降低同类问题重复排查成本
- 让问题处理形成“修复 → 记录 → 推送”的闭环

## 复用方式

以后当用户提出类似请求时，这项能力应自动启用：

- “帮我修一下 OpenClaw 自身的问题”
- “把这次处理记录到 issues”
- “修完顺手推到 GitHub”
- “排查一下 gateway / node / control UI / Telegram”

## 现状

- 已创建本地自定义 skill
- 已补充 AGENTS.md 兜底规则
- 已完成真实场景验证
- 已成功推送到 GitHub issues 仓库
