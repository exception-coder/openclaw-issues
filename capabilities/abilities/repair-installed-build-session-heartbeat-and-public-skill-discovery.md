# 修复 installed build 中的会话 `heartbeat` 误标与 `public` skills 发现缺失

## 能力概述

当 OpenClaw 的 installed build（`dist/`）出现以下两类问题时，可以直接在运行产物层完成定位、热补丁、重载与验收：

1. 主会话被错误显示为 `heartbeat`
2. `~/.openclaw/skills/public/*/SKILL.md` 下的 public skills 无法进入技能菜单 / `openclaw skills list`

这项能力的核心不是“改单个文件碰碰运气”，而是：

- 确认实际运行链对应的多个 dist 产物
- 对所有实际生效副本同步补丁
- reload gateway
- 用 CLI/现场值做闭环验收

## 适用场景

- Control UI 右上角会话列表显示 `heartbeat`
- `sessions.json` 里有主会话，但 UI 显示名被污染
- `openclaw skills list --json` 看不到 `~/.openclaw/skills/public` 下的 skills
- Skills 页面 / 技能菜单搜索不到明明已存在的 public skills

## 方法

### 一、修会话 `heartbeat` 误标

#### 1. 先读现场值

- `~/.openclaw/agents/main/sessions/sessions.json`
- `sessions_list(limit=50, messageLimit=1)`

重点看：

- `agent:main:main`
- `origin.label`
- `origin.from`
- `origin.to`

#### 2. 判断是不是 synthetic heartbeat origin 覆盖主会话

如果出现：

- `provider=webchat`
- `surface=webchat`
- `chatType=direct`
- `label/from/to = heartbeat`

而已有 origin 本应是正常 `webchat`，则需要在 session metadata patch 逻辑里拦截覆盖。

#### 3. 同步补丁所有实际运行产物

不要只改一份。要全文检索并补丁所有实际生效的 dist 副本，例如：

- `sessions-*.js`
- `pi-embedded-helpers-*.js`

#### 4. 修复已污染 store

手工把 `sessions.json` 中主会话 origin 改回正常 `webchat` 值。

#### 5. reload + 再验

- `gateway restart`
- 再看 `sessions.json`
- 再看 `sessions_list(...)`

## 二、修 `public` skills 不被发现

#### 1. 先验证是不是 loader 问题

跑：

- `openclaw skills list --json`
- `openclaw skills info <skill-name> --json`

如果 CLI 都看不到，就不是前端搜索框问题，而是 discovery 问题。

#### 2. 看 `loadSkillEntries(...)`

重点检查 loader 是否只扫：

- `managedSkillsDir`
- `workspaceDir/skills`
- `bundledSkillsDir`
- `extraDirs`

而漏掉：

- `path.join(managedSkillsDir, "public")`

#### 3. 补 loader

在 skills loader 中额外加入：

- `publicManagedSkills = loadSkills({ dir: path.join(managedSkillsDir, "public"), source: "openclaw-managed" })`

并合并到最终 map。

#### 4. reload + 验收

- `gateway restart`
- `openclaw skills list --json`
- `openclaw skills info <skill-name> --json`

## 验收标准

### 会话标签

- `sessions.json` 中主会话不再被写成 `heartbeat`
- `sessions_list(...)` 显示名恢复为 `webchat`

### public skills

- `openclaw skills list --json` 能列出 public skills
- `openclaw skills info <skill-name> --json` 返回 filePath 指向 `~/.openclaw/skills/public/.../SKILL.md`
- Skills 页面/菜单搜索应随之后端结果恢复

## 注意事项

- 这是 installed build 热补丁；升级后可能失效
- 不要只看 UI，要用 CLI 和 session store 双重验收
- 如果一个问题涉及 UI 标签污染 + 列表裁剪，要拆开定位，不要混为一个根因
