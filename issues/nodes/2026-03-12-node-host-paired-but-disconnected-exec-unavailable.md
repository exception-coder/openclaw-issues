# 2026-03-12 · node host 已配对但未连接，导致 exec 不可用

## 元信息

- **分类**：nodes
- **状态**：已定位
- **影响范围**：当前机器 / 本机命令执行
- **相关组件**：nodes / exec / gateway / control-ui
- **相关问题**：
  - [`./2026-03-12-node-service-launchagent-not-loaded-recovered-by-bootstrap-kickstart.md`](./2026-03-12-node-service-launchagent-not-loaded-recovered-by-bootstrap-kickstart.md)

## 问题现象

在需要执行本机命令时，`exec` 无法运行诸如：

- `openclaw doctor --non-interactive`
- `git init`
- `git push`

相关表现：

- `exec host=node requires a node that supports system.run`
- 节点状态查询显示：
  - `paired = true`
  - `connected = false`

## 症状 → 优先排查项 → 修复命令

| 症状 | 优先排查项 | 修复命令/动作 |
|---|---|---|
| `exec host=node requires a node that supports system.run` | 节点是否在线、是否存在 `system.run` node | 检查 nodes 状态与 Control UI 节点页 |
| `paired=true, connected=false` | node host 是否运行、gateway 重启后是否未回连 | 重启 node host / companion 或继续检查 Node service |
| 无法执行 git / doctor | 是否只是 node 断开而不是权限问题 | 优先恢复 node，而不是改 exec 权限 |

## 初步判断

更像是以下之一：

1. node host / companion app 没有运行
2. gateway 重启后，node host 没有自动重连
3. 本机 node host 进程卡住
4. gateway 与 node host 之间连接状态失步

## 检查过程

### 1) 节点状态检查

- 结果：节点记录仍存在
- 结论：不是未配对，而是 **node host 当前不在线或未成功回连 gateway**

### 2) Control UI 节点页检查

- 页面显示：`No nodes with system.run available`
- 结论：问题进一步缩小到“没有在线可执行节点”

## 根因结论

> **node host 已配对，但当前未连接，导致 exec 无法使用。**

这是一个中间层定位结论；后续继续排查后，最终落到 Node service 未加载。

## 采取的修复

### 临时修复

- 打开 Control UI 的节点页确认状态
- 优先重启 node host / companion

### 最终修复

见关联问题：

- [`2026-03-12-node-service-launchagent-not-loaded-recovered-by-bootstrap-kickstart.md`](./2026-03-12-node-service-launchagent-not-loaded-recovered-by-bootstrap-kickstart.md)

## 验证结果

- 本问题单独阶段：已定位
- 后续通过恢复 Node service 解决了 exec 不可用问题

## 风险与回滚

- 风险：如果只看到 exec 报错就误判为权限问题，可能走偏排查方向
- 回滚方式：无；重点是恢复 node 连接

## 后续建议

1. 以后先区分“权限问题”和“节点断开问题”
2. 如果 paired=true 但 connected=false，优先查 Node service

## 可复用经验

### 下次先查什么

1. `paired` / `connected` 状态
2. 是否有 `system.run` 节点可用
3. Node service 是否已加载

### 关键词

- paired connected false
- exec host=node
- system.run unavailable
- node host disconnected
