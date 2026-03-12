# 2026-03-12 · systemEvent 提醒显示像异常，但底层还伴随 embedded agent EOF/timeout 与主会话 lane wait

## 现象

用户观察到：

- 定时任务的监督提示在 UI 中显示得像“请求/系统提示”而不是正常聊天消息
- 怀疑会话异常、无法发消息
- 该问题“经常出现”

本次排查发现这不是单一问题，而是 **两层问题叠加**：

1. **systemEvent 型 cron 的展示语义本来就不是普通消息投递**
2. **运行时还存在 embedded agent 请求偶发 500 EOF / timeout，以及主会话 lane wait 偶发超时**

## 直接结论

### 结论 1：systemEvent cron 并不等于“发聊天消息失败”

本次用于续跑的 cron 是：

- `sessionTarget: main`
- `payload.kind: systemEvent`

这种任务的设计是：

- 向主会话注入一个系统事件
- 驱动 agent 在当前会话继续工作
- **不是**向用户渠道发送一条普通可见消息

因此 UI 上看起来像：

- reminder
- supervision / request-like 提示
- system event

这是符合语义的，不能直接据此判断“消息发送失败”。

## 结论 2：但这台实例确实还存在另一个独立不稳定点

日志里反复出现：

- `embedded run agent end ... error=500 Post "https://chatgpt.com/backend-api/codex/responses": EOF`
- `embedded run agent end ... error=LLM request timed out.`
- `lane wait exceeded: lane=session:agent:main:main waitedMs=29557 queueAhead=1`
- `lane wait exceeded: lane=session:agent:main:main waitedMs=46606 queueAhead=0`

这说明除了 UI 展示语义问题外，还存在：

- 嵌入式 agent 到上游模型请求偶发 EOF / 500
- 有时会退化成 timeout
- 主会话 lane 处理有时会卡到 30~46s

这会让用户体感上更像“会话异常 / 发不出来 / 任务挂住”。

## 本次现场证据

### 1. cron 本身是正常运行的

运行时观察到：

- cron job 创建、arm timer、执行都在正常推进
- 删除 cron 后日志明确显示：
  - `cron: job removed`
  - `cron: armTimer skipped - no jobs with nextRunAtMs`

说明：

- scheduler 不是挂死状态
- job 生命周期本身正常

### 2. UI 的“像请求显示”并不等于 delivery failure

之前 cron list 中能看到：

- `lastRunStatus: ok`
- `lastStatus: ok`
- `lastDeliveryStatus: not-requested`

`not-requested` 很关键，它说明该任务并没有请求聊天投递。

换句话说：

- 它不是“投递失败”
- 而是“本来就没有按普通消息投递”

### 3. 但 embedded agent 请求链路确实不稳定

日志多次出现相同模式：

- `error=500 Post "https://chatgpt.com/backend-api/codex/responses": EOF`
- `error=LLM request timed out.`

这不是 UI 文案问题，而是运行时到上游模型请求链路的真实异常。

### 4. 主 lane 也确实偶发堵塞

日志出现：

- `lane=session:agent:main:main waitedMs=29557`
- `lane=session:agent:main:main waitedMs=46606`

这意味着：

- 就算 scheduler 正常触发
- 主会话的处理队列也可能被上游请求拖慢
- 用户看起来就像“提醒到了，但没正常出消息/没及时响应”

## 影响面

用户体感会混成一个问题：

- reminder 看起来不像正常聊天消息
- 有时又真的会慢、卡、超时
- 最终被感知为“监督提示会话异常/消息发不出去”

所以这个问题不能只按 UI 文案去理解。

## 排查结论（给以后用）

以后再遇到“提醒像异常/像请求显示”时，先分三类判断：

### A. 只是 systemEvent 正常显示

特征：

- cron payload.kind = `systemEvent`
- lastRunStatus = ok
- lastDeliveryStatus = `not-requested`
- 没有明显 embedded agent 报错

结论：

- 这是正常 system event
- 不是聊天消息投递失败

### B. systemEvent + 上游模型请求异常叠加

特征：

- UI 看起来像内部提醒
- 同时日志有：
  - EOF
  - 500
  - timeout
  - lane wait exceeded

结论：

- 展示层和执行层问题叠加
- 需要把根因重点放在 embedded agent / 上游模型请求稳定性

### C. 真正的渠道消息投递问题

特征：

- 任务本来就要求 announce / message delivery
- 但 lastDeliveryStatus 非 ok
- channel probe / provider 状态异常

结论：

- 这才是消息投递层故障

## 建议修复方向

### 短期操作建议

1. **内部续跑任务优先少用高频 systemEvent cron**
   - 尤其在主会话已有较长任务时
   - 2 分钟一次容易和主 lane 堵塞叠加，放大“像异常”的体感

2. **把“后台续跑”与“用户可见通知”分开设计**
   - 续跑：systemEvent
   - 用户通知：announce / message（仅在需要时）

3. **遇到 UI 像异常时先查这三个点**
   - cron list / runs
   - openclaw 日志里的 `agent/embedded`
   - `lane wait exceeded`

### 中期修复建议

1. 在产品/UI里区分展示：
   - `systemEvent reminder`
   - `user-visible delivered message`

2. 在 cron/job 状态页增加更直白字段：
   - `delivery: not-requested (system event only)`

3. 对 repeated EOF/timeout 增加聚合告警，不要只散落在日志里

### 长期建议

把这类“主会话内驱动型 cron”改成更稳的策略：

- 低频触发
- 或独立 isolated agentTurn
- 避免和主会话同 lane 长时间抢占

## 重启后复查补充（16:24 之后）

在用户同意后执行了一次 gateway 重启，目的为：

- 清掉主会话 lane 堵塞态
- 清掉 embedded agent 瞬时异常状态

复查结果：

### 复查现象

- `openclaw doctor --non-interactive` 能正常完成
- `openclaw status` / `openclaw health --json` 均正常返回
- 主会话仍在，Telegram probe 正常
- 当前 cron 已清空

### 复查结论

1. **重启对 lane 堵塞态有缓解作用**
   - 复查后未立刻看到新的 `lane wait exceeded`
   - 说明之前的主会话堵塞态被清掉了

2. **但根因未被根治**
   - 日志仍能确认此前多次出现：
     - `500 ... EOF`
     - `LLM request timed out`
   - 因此更深层不稳定点仍是 embedded agent 到上游模型请求链路

3. **重启过程本身也暴露了一个信号**
   - 日志出现：`shutdown timed out; exiting without full cleanup`
   - 说明运行时在压力/卡顿场景下，关停不总是完全优雅
   - 这不等于重启失败，但属于值得记录的稳定性信号

### 更新后的判断

这类“监督提示像请求显示/像异常”的 case，后续应按四层拆分：

1. `systemEvent` 正常展示语义
2. embedded agent 上游请求 EOF / timeout
3. 主会话 lane 堵塞
4. 重启/关停阶段的 timeout 与非完全清理

## 本次处理动作

- 确认 `Java并发` 核验 cron 已删除
- 当前 cron 列表为空
- 记录此 case 到知识库，后续复现时先按本 issue 排查
- 在用户同意后执行一次 gateway 重启，并补充重启后复查结果

## 快速排查命令

```bash
openclaw status
openclaw health --json
```

查看日志重点关键词：

- `cron`
- `agent/embedded`
- `EOF`
- `timed out`
- `lane wait exceeded`

## 备注

本 case 说明：

> “监督提示长得像异常” 和 “运行时真的有 embedded agent 不稳定” 可能同时存在，不要把两者混为一个原因。
