# 本仓约定

一个仓、扁平 `skills/<name>/SKILL.md`。不要按领域嵌套目录，分类写在 README 表里。

## 新增

1. `npx skills init skills/<name>`，`name` 与目录名一致（小写、连字符）。
2. `description` 写第三人称：做什么 + 何时用。自动触发才省略 `disable-model-invocation`。
3. `SKILL.md` 尽量短；细则放到同目录 `references/`，从正文链过去。
4. 改完更新根 README 的目录表。

## 触发

- 跨项目、不点名也该用的：自动触发（写清触发词）。
- 偶尔用、工具型：加 `disable-model-invocation: true`。
- 产品项目自己的约定（端口、SessionStore）不要进本仓，放那个项目的 `AGENTS.md` 或 `.cursor/skills/`。
