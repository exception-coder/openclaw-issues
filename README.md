# Issues

这里记录 OpenClaw 使用过程中的故障、排查过程、修复动作和复盘结论。

## 目的

- 避免同类问题重复排查
- 形成可检索的故障知识库
- 为后续配置优化和稳定性改进提供依据

## 建议命名方式

文件名建议使用：

- `YYYY-MM-DD-主题.md`

例如：

- `2026-03-12-control-ui-gateway-token-missing-telegram-session-not-visible.md`

## 建议结构

每个问题记录尽量包含：

1. **问题现象**
2. **初步判断**
3. **检查过程**
4. **根因结论**
5. **采取的修复**
6. **修复结果**
7. **后续建议**
8. **可复用经验**

## 当前记录

- [2026-03-12 · Control UI 显示 gateway token missing，Telegram 会话不显示 / 已读不回](./2026-03-12-control-ui-gateway-token-missing-telegram-session-not-visible.md)
- [2026-03-12 · node host 已配对但未连接，导致 exec 不可用](./2026-03-12-node-host-paired-but-disconnected-exec-unavailable.md)
- [2026-03-12 · Node service 未加载，通过 launchctl bootstrap/kickstart 恢复](./2026-03-12-node-service-launchagent-not-loaded-recovered-by-bootstrap-kickstart.md)
