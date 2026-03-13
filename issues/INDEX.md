# OpenClaw Issues Index

这个目录用于沉淀 OpenClaw 使用过程中的故障、排查过程、修复动作与复盘结论。

## 使用方式

- 新问题优先按分类放入子目录
- 文件名统一使用：`YYYY-MM-DD-主题.md`
- 新建问题时优先从 [`TEMPLATE.md`](./TEMPLATE.md) 复制
- 如果一个问题跨多个层面，按**主因**归类，并在文中链接相关问题

## 目录结构

- [`ui/`](./ui/)：Control UI、页面状态、列表显示、认证提示等
- [`channels/`](./channels/)：Telegram / WhatsApp / Discord 等频道收发异常
- [`gateway/`](./gateway/)：Gateway 配置、认证、重启、健康状态
- [`nodes/`](./nodes/)：Node service、配对、连接、system.run、LaunchAgent
- [`sessions/`](./sessions/)：会话显示、绑定、索引、恢复
- [`patterns/`](./patterns/)：通用排查模式、症状到根因的映射

## 快速检索：症状 → 优先排查项

| 症状 | 优先排查项 | 常见修复 |
|---|---|---|
| `gateway token missing` | Control UI 本地设置、gateway auth 模式、页面是否未授权 | 重新保存 token；必要时临时恢复 loopback 无认证后再修回 |
| 会话菜单数量和聊天列表不一致 | UI 缓存、gateway 授权状态、真实 `sessions_list` 返回值 | 刷新 UI，先确认 gateway 已授权连接 |
| Telegram 已读但不回复 | channel 状态、Telegram 会话是否仍存在、gateway/UI 状态 | 先确认 channel 正常，再看会话和 agent 链路 |
| `exec host=node requires a node that supports system.run` | node 是否在线、节点绑定、Node service 状态 | 恢复 node host / Node service |
| `Node service ... not loaded` | macOS LaunchAgent 是否加载 | `launchctl bootstrap` + `launchctl kickstart` |
| `No nodes with system.run available` | 节点是否回连、是否存在可执行 node | 恢复 node service，检查节点绑定 |
| 对外发布动作超时后出现重复发送/重复发帖 | 是否把“超时”误判为“失败”并直接重试；是否混用了多条执行链路 | 先查是否已落地，再决定是否补发；单次发布只保留一条执行链路 |

## 当前问题记录

### UI
- [2026-03-12 · Control UI 显示 gateway token missing，Telegram 会话不显示 / 已读不回](./ui/2026-03-12-control-ui-gateway-token-missing-telegram-session-not-visible.md)
- [2026-03-12 · Control UI 会话被错误显示为 `heartbeat`，且 `~/.openclaw/skills/public` 下 skills 未进入技能菜单](./ui/2026-03-12-control-ui-session-heartbeat-label-corruption-and-public-skills-not-listed.md)

### Gateway
- [2026-03-12 · 新增 `openclaw-self-maintenance` skill，用于固化自身问题处理后的 issues 记录与推送](./gateway/2026-03-12-add-openclaw-self-maintenance-skill-for-issue-capture.md)
- [2026-03-12 · 新增 `docs/capabilities` 目录，用于记录能力与实际案例](./gateway/2026-03-12-add-capabilities-directory-for-ability-and-scenario-tracking.md)

### Channels
- [2026-03-13 · 小红书发布超时后重复触发，导致重复发帖](./channels/2026-03-13-xiaohongshu-publish-timeout-retry-caused-duplicate-posts.md)

### Nodes
- [2026-03-12 · node host 已配对但未连接，导致 exec 不可用](./nodes/2026-03-12-node-host-paired-but-disconnected-exec-unavailable.md)
- [2026-03-12 · Node service 未加载，通过 launchctl bootstrap/kickstart 恢复](./nodes/2026-03-12-node-service-launchagent-not-loaded-recovered-by-bootstrap-kickstart.md)

## 建议维护规则

1. **先记录现象，再记录结论**，避免事后脑补
2. **把命令写成可复制格式**
3. **区分临时修复与最终修复**
4. **写明风险与回滚方式**
5. **补一行“下次先查什么”**，让未来自己少走弯路
