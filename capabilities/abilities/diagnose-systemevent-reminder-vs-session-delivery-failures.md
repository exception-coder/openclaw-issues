# Diagnose systemEvent reminder display vs session delivery failures

## 能力说明

当用户反馈：

- 定时任务/监督提示显示得像“请求”或“系统提示”
- 怀疑会话异常、消息没发出去
- 觉得任务中断或没有继续执行

需要把问题拆成几层，而不是直接归因成“发消息失败”。

## 适用场景

- `cron` 触发的 reminder / supervision / request-like 提示
- `sessionTarget=main` 的内部续跑型任务
- 用户感觉“像异常”“像没发消息”的情况
- 排查 OpenClaw 主会话 lane 卡顿、embedded agent EOF/timeout

## 核心判断框架

### 第一层：先区分任务类型

如果 cron 是：

- `sessionTarget = main`
- `payload.kind = systemEvent`

那它的含义是：

- 向主会话注入系统事件
- 驱动 agent 继续工作
- **不是普通聊天消息投递**

因此 UI 显示成 supervision / request-like / reminder 是正常现象，不能直接判断“消息失败”。

### 第二层：再判断是否有真正执行层异常

重点查日志是否出现：

- `embedded run agent end ... EOF`
- `LLM request timed out`
- `lane wait exceeded: lane=session:agent:main:main ...`
- `shutdown timed out; exiting without full cleanup`

如果有这些，就不是单纯 UI 展示问题，而是执行链路本身也有不稳定。

### 第三层：最后再区分是否为渠道投递故障

只有在任务本来要求 `announce` / 显式投递消息时，才需要重点看：

- `lastDeliveryStatus`
- channel probe
- provider/channel 健康状态

## 推荐排查步骤

1. 查看 cron 状态
   - 确认任务是否是 `systemEvent`
   - 看 `lastRunStatus`
   - 看 `lastDeliveryStatus`

2. 查看 OpenClaw 状态与健康
   - `openclaw status`
   - `openclaw health --json`
   - `openclaw doctor --non-interactive`

3. 查看日志关键词
   - `agent/embedded`
   - `EOF`
   - `timed out`
   - `lane wait exceeded`
   - `shutdown timed out`

4. 判断是否需要重启 gateway
   - 如果主会话 lane 已明显堵塞
   - 或 embedded agent 异常后主会话持续发僵
   - 可先做一次最小扰动的 gateway restart

5. 重启后复查
   - `openclaw status`
   - `openclaw health --json`
   - 日志是否仍持续出现 `lane wait exceeded`

## 实战结论（本次验证）

已验证过的一条真实案例表明：

- supervision/request-like 提示不一定是消息失败
- 但它可能和 embedded agent EOF/timeout、主 lane 堵塞同时出现
- 重启 gateway 可以缓解 lane 堵塞态
- 但不等于根治 embedded agent 上游请求失败

## 处置建议

### 短期

- 少用高频 `systemEvent` cron 去驱动长任务
- 内部续跑和用户可见通知分开设计
- 遇到“像异常”先查执行链路，不要只盯 UI

### 中期

- 为这类 case 建立统一排查文档
- 每次处理后都写入 `docs/openclaw-knowledge/issues`
- 处理完自身问题后，自动 commit + push

### 长期

- 若 EOF / timeout 反复出现，优先排查 embedded agent 上游链路稳定性
- 如果主会话 lane 经常堵塞，评估把长任务从 main session 拆到 isolated 执行

## 相关案例

- `docs/openclaw-knowledge/issues/2026-03-12-systemevent-reminder-display-vs-embedded-agent-eof-timeout.md`
