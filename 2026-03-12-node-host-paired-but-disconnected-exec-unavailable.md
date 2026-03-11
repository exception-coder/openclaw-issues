# 2026-03-12 · node host 已配对但未连接，导致 exec 不可用

## 问题现象

在需要执行本机命令时，`exec` 工具报错，无法运行诸如：

- `openclaw doctor --non-interactive`
- `git init`
- `git push`

相关表现：

- `exec host=node requires a node that supports system.run`
- 节点状态查询显示：
  - `paired = true`
  - `connected = false`

## 影响

当 node host 处于“已配对但断开”的状态时，会导致：

- 无法执行本机 shell 命令
- 无法进行 git 初始化、提交、推送
- 无法直接运行本机诊断命令
- 一些依赖 node host 的自动化能力失效

## 检查结果

通过节点状态检查，确认：

- 节点记录仍存在
- 说明不是“未配对”
- 问题在于：**node host 当前不在线或未成功回连 gateway**

## 初步结论

根因更像是以下之一：

1. node host / companion app 没有运行
2. gateway 重启后，node host 没有自动重连
3. 本机 node host 进程卡住
4. gateway 与 node host 之间连接状态失步

## 处理建议

推荐按以下顺序处理：

1. 打开 Control UI 的节点页面，确认节点状态
2. 重启 node host / companion app
3. 如果仍未恢复，再重启 gateway
4. 恢复后再次检查节点是否变为：
   - `paired = true`
   - `connected = true`

## 可复用经验

如果后续再次看到：

- 节点是 paired，但 connected=false
- exec 无法使用
- 本机命令跑不起来

优先判断为：

> **node host 断开，不是工具权限缺失。**

## 状态

- 当前状态：待用户在本机侧重启/恢复 node host
- 后续动作：node host 恢复连接后，再继续执行 git 初始化与远程推送
