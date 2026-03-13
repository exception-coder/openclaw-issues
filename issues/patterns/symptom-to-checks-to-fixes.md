# 症状 → 排查项 → 修复命令

这个文档记录可复用的排查套路，帮助快速从“表象”缩到“主因”。

## 1. Control UI 报 `gateway token missing`

### 优先排查项

1. Control UI 本地设置里是否缺少 `gatewayToken`
2. gateway 当前 auth 模式是否是 `token`
3. 页面是否处于未授权/离线状态

### 常见修复

- 重新在 Control UI settings 保存 gateway token
- 如需临时抢修本机 loopback 场景，可短时改为：

```json
{
  "gateway": {
    "auth": {
      "mode": "none"
    }
  }
}
```

> 注意：这只是临时修复，后续应恢复 token 认证。

---

## 2. Telegram 已读但不回复

### 优先排查项

1. `openclaw status` 中 Telegram channel 是否为 `OK`
2. 对应 Telegram 会话是否仍存在
3. Control UI / gateway 当前是否在线
4. 是否只是 UI 误判导致“看起来像没回复链路”

### 常见修复

- 先修复 gateway / UI 授权问题
- 再验证 Telegram 会话是否仍在
- 最后才考虑重建 Telegram 连接

---

## 3. `exec host=node requires a node that supports system.run`

### 优先排查项

1. 节点是否 `paired=true` 但 `connected=false`
2. 是否存在可用的 `system.run` node
3. Node service 是否已加载
4. 默认 exec node binding 是否还指向不可用节点

### 常见修复

- 恢复 node host / node service
- 重新检查节点绑定
- 用 `openclaw status` 验证恢复

---

## 4. `Node service ... not loaded`

### 优先排查项

1. macOS LaunchAgent 是否存在
2. LaunchAgent 是否已 bootstrap
3. 对应 service label 是否可 kickstart

### 常见修复命令

```bash
launchctl bootstrap gui/$(id -u) ~/Library/LaunchAgents/ai.openclaw.node.plist
launchctl kickstart -k gui/$(id -u)/ai.openclaw.node
```

然后执行：

```bash
openclaw status
```

确认 `Node service` 不再显示 `not loaded`。

---

## 5. 会话菜单数量和聊天列表不一致

### 优先排查项

1. 当前页面是否未授权或断连
2. 后端真实 `sessions_list` 返回了多少会话
3. UI 是否存在缓存/旧状态

### 常见修复

- 先恢复 gateway 授权
- 再刷新 Control UI
- 不要先假设“会话丢了”

---

## 6. 对外发布/发送动作超时后出现重复内容

### 优先排查项

1. 第一次调用是真的失败，还是仅仅“等待结果超时”
2. 平台侧是否可能已经受理并落地
3. 是否又沿别的执行链路重发了一次（例如 `exec` → `nodes.run` → 后台 `exec`）
4. 当前 skill / 工具是否支持查询最近发布结果

### 常见修复

- 不要把超时直接当失败处理
- 先查询结果，再决定是否补发
- 同一条外部写操作只保留一条执行链路
- 没有幂等能力时，宁可人工确认，也不要自动重试
