# 2026-03-12 · Node service 未加载，通过 launchctl bootstrap/kickstart 恢复

## 问题现象

`openclaw status` 显示：

- `Node service │ LaunchAgent installed · not loaded · unknown`

同时伴随现象包括：

- node host 无法提供 `system.run`
- `exec` 无法在本机执行命令
- Control UI 节点页提示 `No nodes with system.run available`

## 排查结论

问题不在 Telegram，也不在 gateway 主服务本身，而是：

> **Node service 的 LaunchAgent 已安装，但当前没有被加载。**

## 恢复命令

在 macOS 上执行：

```bash
launchctl bootstrap gui/$(id -u) ~/Library/LaunchAgents/ai.openclaw.node.plist
launchctl kickstart -k gui/$(id -u)/ai.openclaw.node
```

## 结果

用户反馈执行后已恢复正常。

## 可复用经验

如果以后 `openclaw status` 再出现：

- `Node service ... not loaded`

优先尝试：

1. `launchctl bootstrap gui/$(id -u) ~/Library/LaunchAgents/ai.openclaw.node.plist`
2. `launchctl kickstart -k gui/$(id -u)/ai.openclaw.node`
3. 再次运行 `openclaw status` 验证恢复

## 状态

- 已恢复
- 后续可继续进行依赖 node host 的本机命令执行与 git 操作
