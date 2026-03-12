# 2026-03-12 · Control UI 会话被错误显示为 `heartbeat`，且 `~/.openclaw/skills/public` 下 skills 未进入技能菜单

## 元信息

- **分类**：ui
- **状态**：已修复（installed build 热补丁）
- **影响范围**：当前机器 / Control UI / sessions / skills loader
- **相关组件**：control-ui / sessions / heartbeat / skills / gateway
- **相关问题**：
  - [`./2026-03-12-control-ui-gateway-token-missing-telegram-session-not-visible.md`](./2026-03-12-control-ui-gateway-token-missing-telegram-session-not-visible.md)
  - [`../patterns/symptom-to-checks-to-fixes.md`](../patterns/symptom-to-checks-to-fixes.md)

## 问题现象

### A. 右上角会话列表被错误显示成 `heartbeat`

- 用户反馈聊天窗口右上角的会话列表只显示一个 `heartbeat`
- `sessions.json` 实际存在两条记录：
  - `agent:main:main`
  - `agent:main:telegram:direct:758510305`
- 但 `sessions_list(limit=50, messageLimit=1)` 一度只返回主会话，且显示名为 `heartbeat`
- 进一步检查发现：主会话 `agent:main:main` 的 `origin.label/from/to` 被反复写成 `heartbeat`

### B. `~/.openclaw/skills/public` 下技能无法在技能菜单搜索到

- 用户在 `~/.openclaw/skills/public/*/SKILL.md` 下创建了 skills
- 这些 skills 文件内容和 frontmatter 正常
- 但 Skills 页面 / 技能菜单无法搜索到
- `openclaw skills list --json` 初始时也只返回 bundled skills
- `openclaw skills info openclaw-self-maintenance --json` / `... yuque-knowledge-summarizer ...` 初始返回 not found

## 检查过程

### 1) 会话列表问题

确认以下事实：

- `~/.openclaw/agents/main/sessions/sessions.json` 包含：
  - `agent:main:main`
  - `agent:main:telegram:direct:758510305`
- 右上角下拉本身不是唯一根因，真正显示值来自 `sessions.list`
- 现场值多次显示主会话被写成：
  - `origin.label = heartbeat`
  - `origin.from = heartbeat`
  - `origin.to = heartbeat`

继续反编译 installed build 后确认：

- 真正运行时存在多份 dist 产物包含同一段 session 元数据写入逻辑
- heartbeat 运行链会把主会话的伪 sender/origin 反写回 session store

### 2) public skills 不显示问题

确认以下事实：

- 本地确实存在：
  - `~/.openclaw/skills/public/openclaw-self-maintenance/SKILL.md`
  - `~/.openclaw/skills/public/yuque-knowledge-summarizer/SKILL.md`
- skills loader 的 `loadSkillEntries(...)` 默认扫描：
  - `managedSkillsDir`（即 `~/.openclaw/skills`）
  - workspace skills
  - bundled skills
  - extra dirs
- 但当前 installed build **没有显式把 `path.join(managedSkillsDir, "public")` 作为一个被发现并合并的 source/root**
- 因此放在 `~/.openclaw/skills/public/*` 的 skills 不会进入最终技能列表

## 根因结论

### A. `heartbeat` 会话误标

> heartbeat 相关运行链会把 `agent:main:main` 的 session origin 元数据回写成 `heartbeat`，导致 Control UI 右上角会话列表错误显示为 `heartbeat`。

### B. public skills 不显示

> skills loader 没有把 `~/.openclaw/skills/public` 下的技能目录显式并入 managed skill discovery，导致按规则放置的 public skills 不会进入 Skills 列表与菜单搜索结果。

## 修复动作

### A. 修复 heartbeat 污染主会话标签

对 installed build 中多份实际运行产物打补丁，拦截 synthetic heartbeat origin 覆盖已有 `webchat` 主会话 origin：

- `dist/sessions-DTVAB3HG.js`
- `dist/sessions-Bh025Q5O.js`
- `dist/pi-embedded-helpers-0rK8Y0KQ.js`
- `dist/pi-embedded-helpers-BxOnIV6d.js`

补丁思路：

- 当新 origin 满足：
  - `provider=webchat`
  - `surface=webchat`
  - `chatType=direct`
  - `label/from/to = heartbeat`
- 且已有 origin 已是正常 `webchat/direct`
- 则保留 existing origin，不让 heartbeat 覆盖主会话标签

同时手工修正已污染的：

- `~/.openclaw/agents/main/sessions/sessions.json`

将主会话恢复为：

- `label: webchat`
- `from: webchat`
- `to: webchat`

之后 reload gateway 让补丁生效。

### B. 修复 public skills 加载链

对 installed build 中多份 skills loader 产物打补丁：

- `dist/skills-B2xU2F7d.js`
- `dist/skills-BcTP9HTD.js`
- `dist/skills-D9zp-Tdj.js`
- `dist/skills-DSroUgpf.js`

补丁动作：

- 在原有 `managedSkillsDir` 加载后，额外加载：
  - `path.join(managedSkillsDir, "public")`
- 并把 `publicManagedSkills` 合并进最终 skills map

之后 reload gateway 让技能加载补丁生效。

## 验证结果

### 会话标签修复结果

- 用户确认：右上角不再只显示 `heartbeat`，`webchat` 已恢复出现
- `sessions.json` 复查主会话 origin 恢复为 `webchat`
- `sessions_list(...)` 返回显示名恢复为 `webchat`

### public skills 修复结果

以下命令修复后已成功返回：

- `openclaw skills list --json`
- `openclaw skills check --json`
- `openclaw skills info openclaw-self-maintenance --json`
- `openclaw skills info yuque-knowledge-summarizer --json`

修复后可见：

- `openclaw-self-maintenance`
  - `source: openclaw-managed`
  - `eligible: true`
  - `filePath: /Users/zhangkai/.openclaw/skills/public/openclaw-self-maintenance/SKILL.md`
- `yuque-knowledge-summarizer`
  - `source: openclaw-managed`
  - `eligible: true`
  - `filePath: /Users/zhangkai/.openclaw/skills/public/yuque-knowledge-summarizer/SKILL.md`

## 风险与说明

- 这是对 installed build `dist/` 的热补丁，不是上游源码级修复
- 后续 OpenClaw 升级后，补丁可能被覆盖，需要重新验证
- `Telegram` 那条会话是否出现在右上角列表，和 `heartbeat -> webchat` 修复是两层不同问题；本次已修好 heartbeat 误标与 public skills discovery，Telegram 会话列表裁剪链若仍异常需单独继续查

## 下次先查什么

### 会话被显示成 heartbeat

1. 直接读 `~/.openclaw/agents/main/sessions/sessions.json`
2. 看 `agent:main:main` 的 `origin.label/from/to` 是否被写成 `heartbeat`
3. 如果被污染，不要只改 `sessions.json`，要同时查 dist 中实际运行的 session metadata 写入链

### public skills 不进技能菜单

1. 先跑：
   - `openclaw skills list --json`
   - `openclaw skills info <skill-name> --json`
2. 如果 CLI 都看不到，就不是前端搜索框问题，而是 loader/discovery 问题
3. 重点检查 `loadSkillEntries(...)` 是否覆盖了 `~/.openclaw/skills/public`

## 关键词

- session list heartbeat
- webchat shown as heartbeat
- heartbeat origin label corruption
- public skills not listed
- ~/.openclaw/skills/public not discovered
- skills menu cannot search custom skill
