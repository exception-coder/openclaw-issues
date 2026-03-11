# 2026-03-12 · Control UI 显示 `gateway token missing`，Telegram 会话不显示 / 已读不回

## 元信息

- **分类**：ui
- **状态**：已临时修复
- **影响范围**：当前机器 / Control UI / Telegram 观察链路
- **相关组件**：control-ui / gateway / telegram / sessions
- **相关问题**：
  - [`../nodes/2026-03-12-node-host-paired-but-disconnected-exec-unavailable.md`](../nodes/2026-03-12-node-host-paired-but-disconnected-exec-unavailable.md)

## 问题现象

- 会话菜单里像是有 2 个会话，但聊天列表里只看到 `Main Session`
- 用户怀疑此前配置的 Telegram 会话丢失
- Telegram 中给机器人发消息显示“已读”，但没有回复
- Control UI 页面出现：
  - `unauthorized: gateway token missing (open the dashboard URL and paste the token in Control UI settings)`
- 页面同时显示：
  - 网关离线
  - 已断开与网关的连接

## 症状 → 优先排查项 → 修复命令

| 症状 | 优先排查项 | 修复命令/动作 |
|---|---|---|
| `gateway token missing` | 检查 Control UI 本地设置是否缺少 `gatewayToken` | 重新保存 token；抢修时可临时将 `gateway.auth.mode` 改为 `none` |
| 会话菜单和聊天列表不一致 | 检查页面是否未授权、后端真实会话数是多少 | 先恢复 gateway/UI 授权，再刷新页面 |
| Telegram 已读但不回 | 先确认 Telegram 配置、会话和 UI/gateway 状态 | 不要先重建 Telegram，先修 UI/gateway |

## 初步判断

先区分两个层面：

1. Telegram 配置是否丢失
2. Control UI 是否失去对 gateway 的认证/连接

本次更核心的问题不是 Telegram 配置被删除，而是 **Control UI 本地保存的 gateway token 丢失**，导致 UI 未授权、断连、会话列表异常。

## 检查过程

### 1) 检查后端真实会话列表

- 结果：后端当时只返回 `agent:main:main`
- 结论：当时 UI 中的“第二个会话”更像旧状态或缓存

### 2) 检查 Telegram 配置是否还在

确认以下项目仍存在：

- `channels.telegram.enabled = true`
- `channels.telegram.dmPolicy = pairing`
- `channels.telegram.allowFrom` 存在
- `channels.telegram.botToken` 存在（已脱敏）
- `channels.telegram.proxy = http://127.0.0.1:7890`
- `plugins.entries.telegram.enabled = true`

结论：**Telegram 配置没有丢**。

### 3) 检查 Control UI 页面状态

页面明确显示：

- `unauthorized: gateway token missing`
- `已断开与网关的连接`
- `健康状况：离线`

结论：UI 当时没有正确连上 gateway，页面状态不可信。

### 4) 检查 Control UI 本地存储

发现 `openclaw.control.settings.v1` 中存在：

- `gatewayUrl`
- `sessionKey`
- 主题等 UI 设置

但**没有 `gatewayToken`**。

结论：不是 Telegram 配置丢了，而是 **Control UI 本地 token 丢了或未保存成功**。

## 根因结论

> **Control UI 丢失了本地保存的 `gatewayToken`，导致未授权连接 gateway。**

连带后果包括：

- UI 显示离线
- 会话列表异常
- 用户误以为 Telegram 会话丢失

## 采取的修复

### 临时修复

将本机 gateway 的认证从 `token` 临时改为 `none`：

```json
{
  "gateway": {
    "auth": {
      "mode": "none"
    }
  }
}
```

说明：

- 当前 gateway `bind = loopback`
- 仅监听本机回环地址
- 这是**恢复可用性优先**的临时修复，不是最终安全方案

### 最终修复（待完成）

- 恢复 `gateway.auth.mode = token`
- 将正确 token 重新保存到 Control UI settings
- 验证刷新后不再出现 `gateway token missing`

## 验证结果

- gateway 成功重启并应用配置
- Control UI 恢复连接
- 用户确认“现在修复了”
- 后续 `openclaw status` 也确认 Telegram channel 和 Telegram 会话均仍存在

## 风险与回滚

- 风险：当前 gateway 处于本机 loopback 无 token 模式
- 回滚方式：恢复 `gateway.auth.mode = token` 并重新在 Control UI 保存 token

## 后续建议

1. 尽快恢复 gateway token 认证
2. 以后遇到 Telegram 已读不回，先查 UI/gateway 是否失联
3. 不要在 UI 异常时先假设“Telegram 会话丢了”

## 可复用经验

### 下次先查什么

1. 页面是否未授权 / 离线
2. `gatewayToken` 是否保存在 Control UI 本地设置中
3. 后端真实会话列表与 Telegram 配置是否仍存在

### 关键词

- gateway token missing
- control ui unauthorized
- telegram session not visible
- telegram read but no reply
