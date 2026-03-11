# OpenClaw Issues

这是 OpenClaw 的问题记录与排查知识库。

## 入口

- 总索引：[`INDEX.md`](./INDEX.md)
- 故障模板：[`TEMPLATE.md`](./TEMPLATE.md)

## 分类目录

- [`ui/`](./ui/)
- [`channels/`](./channels/)
- [`gateway/`](./gateway/)
- [`nodes/`](./nodes/)
- [`sessions/`](./sessions/)
- [`patterns/`](./patterns/)

## 使用约定

1. 新问题优先按分类放入子目录
2. 文件名统一使用 `YYYY-MM-DD-主题.md`
3. 新建问题优先从 `TEMPLATE.md` 复制
4. 记录时优先写：
   - 现象
   - 检查项
   - 根因
   - 修复命令
   - 风险与回滚

## 当前重点

当前已沉淀的高价值问题集中在：

- Control UI 认证/显示异常
- Node host / Node service 未连接或未加载
- system.run 不可用导致 exec 失效

建议优先从 `INDEX.md` 检索。