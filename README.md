# my-skills

个人 Agent Skills 目录。Git 是源，本机用 `npx skills` 装到 Cursor / Claude。

远程：[ATQQ/skills](https://github.com/ATQQ/skills)

## 目录

| Skill | 做什么 | 触发 |
| --- | --- | --- |
| [`bookkeeper-persona`](./skills/bookkeeper-persona/SKILL.md) | 为妙蛙记账生成自定义人设（名称、介绍、开场白、提示词） | 自动：记账人设、立人设、助手性格 |

## 安装

```bash
# 全仓
npx skills add ATQQ/skills -g

# 只装一个人设 skill
npx skills add ATQQ/skills@bookkeeper-persona -g

# 还没 push 时，从本地装
npx skills add ~/Documents/github/my-skills -g
```

开发期也可软链到 Cursor 个人目录（本仓 `skills/` 是扁平的）：

```bash
ln -s ~/Documents/github/my-skills/skills ~/.cursor/skills
```

## 新增 skill

```bash
npx skills init skills/my-new-skill
```

约定见 [AGENTS.md](./AGENTS.md)。
