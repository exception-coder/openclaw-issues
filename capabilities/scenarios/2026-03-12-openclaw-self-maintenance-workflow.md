# 2026-03-12 · 利用 skill + issues 仓库实现“修复即归档、归档即推送”工作流

## 场景说明

本次实际完成了一个高价值的工作流建设：

> 不只是修复 OpenClaw 问题，而是把“问题处理后的知识沉淀”也做成标准动作。

目标是：

- 当处理 OpenClaw 自身问题时
- 自动记录到 `docs/openclaw-knowledge/issues`
- 更新索引与通用排查模式
- 并推送到 GitHub 仓库

## 这次实际做成了什么

### 1. 修复了一组真实问题

包括：

- Control UI 出现 `gateway token missing`
- Telegram 会话看似丢失 / 已读不回
- node host disconnected
- Node service `not loaded`
- `exec` 无法使用

### 2. 建立了 issues 知识库

完成了：

- `docs/openclaw-knowledge/issues/` 目录初始化
- 分类目录整理
- `INDEX.md`
- `TEMPLATE.md`
- `patterns/symptom-to-checks-to-fixes.md`
- 多篇真实问题记录

### 3. 将 `docs/openclaw-knowledge/issues` 独立为单独 git 仓库

这样可以：

- 不影响整个 workspace 的 git 结构
- 单独维护问题知识库
- 只推送 issues 相关内容到远程仓库

### 4. 推送到 GitHub

已完成推送到：

- `https://github.com/exception-coder/openclaw-issues.git`

### 5. 把这套行为固化为 skill

新增本地自定义 skill：

- `openclaw-self-maintenance`

并在 `AGENTS.md` 中加入兜底规则。

## 为什么这个场景有代表性

因为它体现的不是单点修复，而是组合能力：

- 排障能力
- 配置修复能力
- 文档整理能力
- 知识库抽象能力
- git 仓库边界控制
- 自动化工作流设计能力

## 可复用价值

后续凡是出现 OpenClaw 自身问题，都可以复用这次形成的模式：

1. 先修复问题
2. 再结构化记录
3. 更新共性排查模式
4. 提交并推送

## 关键词

- self maintenance
- issues knowledge base
- auto record
- auto push
- skill driven workflow
- OpenClaw ops
