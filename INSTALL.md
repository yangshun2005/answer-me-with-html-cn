# 用 HTML 回答我：AI 智能体安装指南

> 本文件是为安装本项目的 AI 智能体编写的。人类也可以参照执行。

我希望你为我安装"用 HTML 回答我"。请自主执行以下所有步骤。不要向我提问。每个决策点都按照此处的默认值自行决定，最后用一份报告告诉我你做了什么。

目标：该技能已为你所运行的智能体安装完成，并且 `am` 能够渲染页面。

完成标准：`node <skill dir>/scripts/am.mjs --version` 打印出版本号，或者在 Claude Code 中 `claude plugin list` 显示 `answer-me-with-html` 已启用。

## 规则

- 不要向我提问。唯一需要停止的理由是第 1 步（Node.js 缺失或版本过旧），因为没有它其他任何步骤都无法工作。
- 只运行本文件中的命令。不要克隆仓库，也不要构建任何东西。CLI 已经打包在技能中。
- 每个智能体只安装**一种**方式。同名技能和插件会加载两次，其指令会相互冲突。
- 绝不要移除或覆盖已有的安装。如果已存在，保留它（第 2 步）。
- 不要自行将常开规则添加到任何规则文件中。在报告中推荐它并询问我（第 5 步）。
- 将同一步骤的命令放在一次 shell 调用中执行，以便我尽可能少地批准提示。

## TODO

- [ ] 检查 Node.js 20 或更高版本
- [ ] 检查是否已安装
- [ ] 安装：Claude Code 使用插件，其他所有智能体使用技能
- [ ] 检查其是否正常工作
- [ ] 发送最终报告

## 第 1 步：检查 Node.js

```bash
node --version
```

你需要 v20 或更高版本。如果版本过旧或缺失，请在此停止并用一句话告诉我。不要自行安装 Node.js。

## 第 2 步：检查是否已有安装

```bash
ls -d ~/.agents/skills/answer-me-with-html ~/.claude/skills/answer-me-with-html 2>/dev/null; claude plugin list 2>/dev/null | grep answer-me-with-html
```

无需询问，自行判断：

- 未找到任何内容：转到第 3 步。
- 找到了，且正是第 3 步会为你安装的方式：更新它，然后转到第 4 步。技能使用 `npx -y skills update answer-me-with-html -y`，插件使用 `claude plugin update answer-me-with-html@answer-me-with-html`。
- 找到了，但是不同的方式（例如你在 Claude Code 中，但安装的是技能）：保留它，跳过第 3 步，转到第 4 步。在报告中说明已安装的是哪种方式，以及另一种方式会额外提供什么。

## 第 3 步：安装

### Claude Code

该插件包含技能，还包含 `/answer-me-with-html:config` 和 `/answer-me-with-html:clean` 命令。

```bash
claude plugin marketplace add QingYunA/answer-me-with-html
claude plugin install answer-me-with-html@answer-me-with-html
```

插件会在下一个会话中加载，或者在我运行 `/reload-plugins` 之后加载。请在报告中说明这一点。

### 任何其他智能体（Codex、Cursor、OpenCode 等）

```bash
npx -y skills add QingYunA/answer-me-with-html -g -y -a <your agent name>
```

你知道自己是哪个智能体。使用 `npx -y skills add --help` 中为其记录的名称。对于 Claude Code，名称是 `claude-code`。如果安装程序不知道你的名称，不要猜测，也不要问我：停止此步骤，并将错误写入报告，同时附上 README 中的"Manual install"链接。

此方式仅安装技能。它的工作方式相同，但没有斜杠命令。设置通过 `/answer-me-with-html config`（注意有空格）更改，或在终端中使用 `am config`。Codex 和 Cursor 有自己的插件系统，但本仓库尚未为它们提供插件。不要尝试安装。

## 第 4 步：检查其是否正常工作

对于技能安装：

```bash
dir=$(ls -d ~/.agents/skills/answer-me-with-html ~/.claude/skills/answer-me-with-html 2>/dev/null | head -1)
node "$dir/scripts/am.mjs" --version
```

Claude Code 将技能保存在 `~/.claude/skills` 中；大多数其他智能体读取 `~/.agents/skills`。第一行会找到存在的那个。然后使用该技能制作一个解释 TCP 三次握手的页面，使用 `--no-open` 以避免弹出浏览器窗口。

对于插件安装，技能会在重新加载后加载。运行 `claude plugin list` 并检查其是否已启用。不要等待重新加载。

如果检查失败，请参阅故障排除，尝试修复一次，并报告发生了什么。

## 第 5 步：最终报告

发送一条消息。保持为以下几行：

1. 你安装了什么，以及安装方式（插件或技能）。
2. 检查的结果，以及如果你制作了 TCP 页面，其路径。
3. 如果还有需要我做的事情（例如 `/reload-plugins`），请说明。如果没有，就说"没有"。
4. 如何更改设置：插件使用 `/answer-me-with-html:config`，技能使用 `/answer-me-with-html config`。
5. 一行说明：我们推荐常开模式，它会为每个回答附加一个带有结论的小页面。询问我是否需要它，不要自行添加。该规则位于 [README.md](README.md) 中的"Always-on mode"部分。

## 故障排除

| 问题 | 原因与修复 |
| :--- | :--- |
| `Unknown command: /answer-me-with-html:config` | 插件未安装。请使用 `/answer-me-with-html config`。 |
| 安装后找不到技能 | 启动一个新会话。在 Claude Code 中，运行 `/reload-plugins`。 |
| 相同的指令出现两次 | 技能和插件都已安装。请报告；不要移除任何一个。 |
| `node: command not found`，或 Node 版本低于 20 | 在第 1 步停止并报告。 |
| `npx` 无法访问网络 | 报告，并附上 README 中的"Manual install"链接。 |

立即执行：完成上述 TODO 列表以实现：`am --version` 打印出版本号，且智能体能够看到该技能。不要向我提问；发送最终报告。