# my-skills

自己写的 skills 合辑。每个技能一个子目录，放在 `skills/` 下。

| 技能 | 说明 |
|---|---|
| herdr-subagents | herdr 子代理委派工作流：分屏、agent start/prompt/read、并行派工、干预与清理 |

## 跨客户端使用

技能本体是标准 [Agent Skills](https://agentskills.io) 格式（`SKILL.md`），任何实现该规范的客户端都可用。仓库根的 `package.json` 只服务于 pi 的包安装，其他客户端直接取 `skills/` 下的技能目录：

| 客户端 | 安装方式 |
|---|---|
| pi | `pi install git:github.com/fizecoding/my-skills` |
| Claude Code | 拷贝或链接 `skills/<name>` 到 `~/.claude/skills/`（项目级 `.claude/skills/`） |
| Codex CLI | 拷贝或链接 `skills/<name>` 到 `~/.agents/skills/`（项目级 `.agents/skills/`） |
| 其他实现规范的客户端 | 参照其文档的技能目录 |

以 Claude Code 为例：

```bash
git clone https://github.com/fizecoding/my-skills
ln -s "$(pwd)/my-skills/skills/herdr-subagents" ~/.claude/skills/herdr-subagents
```

不想安装也可以在 pi 里临时试一次：

```bash
pi -e ./my-skills
```

## 新增技能

在 `skills/` 下新建 `<name>/SKILL.md`（frontmatter 至少含 `name` 和 `description`），无需改 package.json —— pi 会从约定的 `skills/` 目录自动发现。
