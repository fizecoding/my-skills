---
name: herdr-subagents
description: 在 herdr 中把任务委派给独立 coding agent 子代理的工作流：分屏、agent start 启动、prompt 派发、read 回收、并行派工、干预与清理。当主会话需要并行探索/实现/review/测试等委派工作且运行于 herdr 环境（HERDR_ENV=1）时使用。与 herdr 技能（原始面板控制）互补；若需要更全的 herdr 控制细节且 herdr 技能未加载，先运行 herdr --skill。
---

# herdr 子代理工作流

主 pi 会话把工作委派给在 herdr pane 中独立运行的 coding agent（子代理）：一个 pane 一个 agent，经 herdr CLI 派发任务、等待完成、读回结果、用后清理。子代理有自己的完整上下文和工具，与主会话仅通过 prompt 文本和终端输出通信。

## 前置条件

只有 pane 的交互主 agent（pi 或 claude）可以使用本技能。子代理（`sub:*` tab、child session、继承环境变量的 headless 进程）不得做任何 herdr 控制：直接用自己的工具完成任务，失败则汇报后停止。

```bash
test "${HERDR_ENV:-}" = 1        # 失败则说明不在 herdr 内，停止
```

herdr CLI 是命令语法的唯一权威，命令组帮助用 `herdr agent`、`herdr pane` 查看（不要裸跑 `herdr`，会启动 TUI）。大多数控制命令返回 JSON：从响应里解析 pane/agent id，不要预测。

## 单个任务的标准循环

### 1. 分屏

默认在当前 tab 分一个兄弟 pane，保持调用者 cwd，不动用户焦点：

```bash
herdr pane split --current --direction right --cwd "$PWD" --no-focus
```

- 方向：宽 pane 向右分，窄/高 pane 向下分（可用 `herdr pane layout --pane "$HERDR_PANE_ID"` 判断）；避免同方向反复分出过窄的列
- 从 `.result.pane.pane_id` 读新 pane ID，后续全部用它

### 2. 启动子代理

pane 必须停在交互 shell 提示符。给它一个有用的唯一名字（`[a-z][a-z0-9_-]{0,31}`，在存活 agent 中唯一；agent 退出后名字释放）：

```bash
herdr agent start scout --kind pi --pane <pane-id>
```

- `--kind` 按 pi/claude/codex/gemini/… 安装情况选；agent 原生参数放 `--` 之后
- 成功即表示 agent 已在同 pane 被检测且可交互；启动期被 block 时返回 `agent_not_ready`，但名字仍可用 `agent read` / `agent send-keys`，等 idle 再派工
- 就绪等待默认 30s，可 `--timeout <ms>` 调整（上限 300000）

### 3. 派发任务

```bash
herdr agent prompt scout "任务描述" --wait --timeout 300000
```

任务书要一次给全：子代理没有本会话的上下文。写清目标、涉及文件路径、约束（如只读/不改哪些）、验收标准和回复格式。

- `--wait` 等提交后第一个 settled 状态（idle/done/blocked），`--timeout <ms>` 必须给，避免无限等
- agent 正在审批/提问对话框时提交会被 `agent_blocked` 拒绝：先读屏，需要人决策的问用户
- 超时或 stalled 不代表没送达：先 `agent read` 判断实际进度，再决定续等还是干预，不要盲目重发

### 4. 回收结果

```bash
herdr agent read scout --source recent-unwrapped --lines 200
```

- 优先 `recent-unwrapped`（软换行已合并，适合读 transcript）；需要颜色/样式作证据时 `--format ansi`
- 输出太长读不全时，让子代理把完整结果写成临时目录下的 Markdown 并只回复文件路径，然后本会话直接读该文件——这是回退手段，不要在首个任务书里就要求写文件
- `herdr agent get scout` 查生命周期状态（idle/working/blocked/done/unknown）；状态只是粗粒度参考，以读到的内容为准

### 5. 清理

任务完成、结果已读、无后续追问且用户没要求保留时，关掉自己创建的 pane：

```bash
herdr pane close <pane-id>
```

## 并行派工

多个子代理分工（如 scout 只读探查、reviewer 只读审查、fixer 实现、tester 写测试）同时跑：

1. 逐个走「分屏 + agent start」拉起 N 个子代理
2. **先全部提交，再逐个等待**——`prompt --wait` 是阻塞的，并行时不要带：

```bash
herdr agent prompt scout "探查任务A…" --timeout 300000      # 不带 --wait，立即返回
herdr agent prompt fixer "实现任务B…" --timeout 300000
herdr agent wait scout  --timeout 300000                     # 再逐个等 settled
herdr agent wait fixer  --timeout 300000
```

3. `herdr agent list` 随时总览全部 agent 状态
4. 全部完成后按各自结果决定追问或逐个 `pane close`

依赖关系排成波次：有先后依赖的任务等上一波全部 settled 再派下一波。

## 运行中干预

```bash
herdr agent send-keys scout esc        # 打断当前生成
herdr agent send-keys scout ctrl+c     # 强中断
herdr agent prompt scout "补充指示…"    # 不带 --wait，作为下一轮输入
```

干预前先 `agent read` 看屏幕现状，别凭记忆操作。`blocked` 状态多半是审批/提问 UI：读屏确认内容，权限类决策问用户，不要替用户批准。

## 铁律

- 一切命令显式指定 pane ID 或 agent 名，永不依赖当前焦点；`--no-focus` 保持用户焦点
- 只关闭自己创建的 pane/tab，绝不碰用户或别的会话的
- 不新建 workspace；子代理与主会话同 tab/同 workspace，cwd 继承调用者
- id 从 JSON 响应解析；`herdr agent list` 确认存活，gone 的 agent 不复用名字假设
- 结果以读到 的内容为准，不靠状态推断完成
