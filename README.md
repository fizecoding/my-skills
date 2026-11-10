# my-skills

自己写的 pi skills 合辑，以 pi package 形式发布。每个技能一个子目录，放在 `skills/` 下。

| 技能 | 说明 |
|---|---|
| herdr-subagents | herdr 子代理委派工作流：分屏、agent start/prompt/read、并行派工、干预与清理 |

## 安装

```bash
pi install git:<repo-url>
# 或发布 npm 后
pi install npm:my-skills
```

不想安装可以临时试一次：

```bash
pi -e ./my-skills
```

## 新增技能

在 `skills/` 下新建 `<name>/SKILL.md`（frontmatter 至少含 `name` 和 `description`），无需改 package.json —— pi 会从约定的 `skills/` 目录自动发现。
