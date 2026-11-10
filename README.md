# my-skills

自己写的 skills 合辑。每个技能一个子目录，放在 `skills/` 下，pi 会从约定目录自动发现并注册为 `/skill:<name>` 命令。

| 技能 | 说明 |
|---|---|
| herdr-subagents | herdr 子代理委派工作流：分屏、agent start/prompt/read、并行派工、干预与清理 |
| openspec-grill-proposal | 访谈式打磨出完整可实施的 OpenSpec 变更（proposal/specs/design/tasks）：双 scout 侦察 → 分轮 grill → 并行评审+起草 → 校验交付；依赖 openspec CLI（项目需已初始化 OpenSpec 工作区）；随包附 openspec-scout / codegraph-scout 两个只读子代理（pi 专属） |

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

注：`openspec-grill-proposal` 依赖 openspec CLI（需已初始化的 OpenSpec 工作区）。全流程在无子代理能力的客户端可降级运行（主会话自任侦察/评审/起草）；`agents/` 下的双 scout 与并行轨道是 pi 安装后的增强，仅随 `pi install` 生效。

不想安装也可以在 pi 里临时试一次：

```bash
pi -e ./my-skills
```

## 新增技能

在 `skills/` 下新建 `<name>/SKILL.md`（frontmatter 至少含 `name` 和 `description`），无需改 package.json —— pi 会从约定的 `skills/` 目录自动发现。
