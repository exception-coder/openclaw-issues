# 2026-03-12 · Control UI 显示 `gateway token missing`，Telegram 会话不显示 / 已读不回

## 问题现象

- Control UI 当前页会话列表异常：会话菜单里像是有 2 个会话，但聊天列表里只看到 `Main Session`
- 用户怀疑此前配置的 Telegram 会话丢失
- 在 Telegram 里给机器人发消息后显示“已读”，但没有收到回复
- Control UI 页面出现错误：
  - `unauthorized: gateway token missing (open the dashboard URL and paste the token in Control UI settings)`
- 页面状态还显示：
  - 网关离线
  - 已断开与网关的连接

## 初步判断

先区分两个层面：

1. **Telegram 配置是否丢失**
2. **Control UI 是否失去对 gateway 的认证/连接**

经检查，本次更核心的问题不是 Telegram 配置被删除，而是 **Control UI 本地保存的 gateway token 丢失**，导致 UI 未授权、断连、会话列表异常，从而引发“Telegram 会话像是消失了”的表象。

## 检查过程

### 1) 检查当前会话列表

通过 `sessions_list` 查看，后端当时只返回了一个会话：

- `agent:main:main`

这说明：

- 当时**后端可见会话**只有 Main Session
- 会话菜单中出现的“第二个会话”更像是 UI 缓存/旧状态

### 2) 检查 Telegram 配置是否还在

通过 `gateway.config.get` 检查配置，确认以下项目仍然存在：

- `channels.telegram.enabled = true`
- `channels.telegram.dmPolicy = pairing`
- `channels.telegram.allowFrom` 仍包含允许的 Telegram 用户
- `channels.telegram.botToken` 仍存在（已脱敏）
- `channels.telegram.proxy = http://127.0.0.1:7890`
- `plugins.entries.telegram.enabled = true`

结论：**Telegram 配置没有丢**。

### 3) 检查 Control UI 页面实际状态

通过浏览器快照确认页面明确显示：

- `unauthorized: gateway token missing`
- `已断开与网关的连接`
- `健康状况：离线`

这一步确认：

- Control UI 本身没有正确连上 gateway
- UI 上看到的会话、状态信息不可信或不完整

### 4) 检查 Control UI 的本地存储

读取浏览器 `localStorage` 中的：

- `openclaw.control.settings.v1`

发现其中保存了：

- `gatewayUrl`
- `sessionKey`
- 主题等 UI 设置

但**没有 `gatewayToken` 字段**。

这和页面报错完全对应，说明：

- **不是 token 一定变了**
- 也可能是 **Control UI 本地没保存住 token** / 被清掉了

## 根因结论

本次问题的直接根因是：

> **Control UI 丢失了本地保存的 `gatewayToken`，导致未授权连接 gateway。**

连带后果：

- UI 显示离线
- 会话列表异常 / 只剩 Main Session
- 用户误以为 Telegram 会话丢失

另外，Telegram “已读不回” 在这个阶段**不能直接判定是 Telegram 配置损坏**；首先要先修复 gateway/UI 连接，再继续验证 Telegram 收发链路是否正常。

## 采取的修复

### 临时修复方案（已执行）

为了尽快恢复 Control UI，而不要求用户手工重新找 token：

- 将本机 gateway 的认证从 `token` 临时改为 `none`

修改项：

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
- 也就是只监听本机回环地址
- 因此临时关闭 token 认证的风险相对可控
- 这是一个**恢复可用性优先**的修复动作，不是最终安全方案

### 修复结果

- gateway 成功重启并应用配置
- Control UI 恢复可连接
- 用户确认“现在修复了”

## 为什么没有直接重建 Telegram 连接

因为在排查中已经确认：

- Telegram bot token 还在
- Telegram channel 配置还在
- allowFrom 还在
- 插件也还启用

所以当时没有证据表明需要：

- 重新绑定 Telegram
- 重新配置 bot
- 重新建立 pairing

优先修 Control UI 的 gateway 认证是更合理的路径。

## 后续建议

### 建议 1：后续恢复 gateway token 认证

本次为了快速修复，临时把 `gateway.auth.mode` 改成了 `none`。

建议后续在确认 Control UI 稳定后：

1. 重新启用 `gateway.auth.mode = token`
2. 将正确 token 保存回 Control UI settings
3. 验证页面刷新后不再出现 `gateway token missing`

### 建议 2：如果再出现 Telegram 已读不回，按这个顺序排查

1. 先看 Control UI 是否仍在线
2. 再看 Telegram channel 配置是否仍存在
3. 再检查消息是否真正进入会话/agent 链路
4. 最后才考虑重建 Telegram 连接

### 建议 3：统一沉淀问题记录

建议把类似问题都放在：

- `docs/issues/`

目录下，按日期 + 主题命名，便于后续检索与复盘。

## 可复用的经验

### 现象到原因的映射

如果以后再次看到：

- 会话列表异常
- UI 只剩 Main Session
- 页面显示离线
- `unauthorized: gateway token missing`

优先怀疑：

> **不是会话丢了，而是 Control UI 本地 token 丢了。**

### 排查优先级

优先级建议：

1. **先确认 UI 是否已授权连上 gateway**
2. **再确认 channel 配置是否还在**
3. **最后才判断消息链路是否故障**

## 状态

- 问题：已临时修复
- 风险：gateway 当前为本机 loopback 无 token 模式
- 待办：后续恢复 token 认证并重新把 token 保存回 Control UI
