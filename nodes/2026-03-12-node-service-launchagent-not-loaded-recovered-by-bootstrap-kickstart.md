# 2026-03-12 · Node service 未加载，通过 launchctl bootstrap/kickstart 恢复

## 元信息

- **分类**：nodes
- **状态**：已彻底修复
- **影响范围**：当前机器 / 本机命令执行 / node host 连接
- **相关组件**：node-service / launchctl / exec / nodes
- **相关问题**：
  - [`./2026-03-12-node-host-paired-but-disconnected-exec-unavailable.md`](./2026-03-12-node-host-paired-but-disconnected-exec-unavailable.md)

## 问题现象

`openclaw status` 显示：

- `Node service │ LaunchAgent installed · not loaded · unknown`

同时伴随现象包括：

- node host 无法提供 `system.run`
- `exec` 无法在本机执行命令
- Control UI 节点页提示 `No nodes with system.run available`

## 症状 → 优先排查项 → 修复命令

| 症状 | 优先排查项 | 修复命令/动作 |
|---|---|---|
| `Node service ... not loaded` | LaunchAgent 是否已安装但未加载 | `launchctl bootstrap` + `launchctl kickstart` |
| `No nodes with system.run available` | Node service 是否真正运行 | 先恢复 Node service，再看节点页 |
| exec 无法使用 | 是否由 Node service 未加载引起 | 恢复 Node service 后复查 `openclaw status` |

## 初步判断

问题不在 Telegram，也不在 gateway 主服务本身，而是：

> **Node service 的 LaunchAgent 已安装，但当前没有被加载。**

## 检查过程

### 1) `openclaw status` 检查

- 结果：`Node service` 行明确显示 `not loaded`
- 结论：Node service 没有进入运行状态

### 2) 节点页检查

- 结果：显示 `No nodes with system.run available`
- 结论：没有可用执行节点，和 `Node service not loaded` 一致

## 根因结论

> **macOS 上的 OpenClaw Node service LaunchAgent 未加载，导致 node host 不可用。**

## 采取的修复

### 最终修复

在 macOS 上执行：

```bash
launchctl bootstrap gui/$(id -u) ~/Library/LaunchAgents/ai.openclaw.node.plist
launchctl kickstart -k gui/$(id -u)/ai.openclaw.node
```

说明：

- 第一条命令负责加载 LaunchAgent
- 第二条命令负责强制拉起对应 service

## 验证结果

- 用户反馈执行后已恢复正常
- 后续 `exec` 已可再次使用
- 本机 git 操作与远程推送也恢复可执行

## 风险与回滚

- 风险：plist 路径或 service label 若与本机实际不一致，命令会失败
- 回滚方式：如需卸载可使用对应 `launchctl bootout`，但本问题场景不需要回滚

## 后续建议

1. 遇到 node/exec 问题先跑 `openclaw status`
2. 如果看到 `Node service ... not loaded`，优先走 LaunchAgent 恢复
3. 可以补一份本机常用恢复命令速查表

## 可复用经验

### 下次先查什么

1. `openclaw status` 中的 `Node service`
2. Control UI 节点页是否显示 `No nodes with system.run available`
3. LaunchAgent 是否已 bootstrap / kickstart

### 关键词

- node service not loaded
- launchctl bootstrap
- launchctl kickstart
- no nodes with system.run available
