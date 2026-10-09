# 设置、清理与更新

当 SKILL.md 第 0 节将你引导至此，请阅读本文。

当本次调用的参数（见 SKILL.md 第 0 节）以 `config` 开头时（例如 `/answer-me-with-html config open off`），本轮仅处理设置，不生成页面：

- `config`：运行 `am config` 显示当前设置，然后让用户选择。如果 agent 有选择工具（如 AskUserQuestion），请使用它：一次最多 4 项设置，优先展示这些：`open`、`theme`、`mode`、`style`，并在选项中标注当前值。否则用纯文本询问。
- `config <key> <value>`：运行 `am config set <key> <value>`。
- `config reset [key]`：运行 `am config reset [key]`。

当用户用自然语言提出请求时（如"不要自动打开浏览器"、"默认使用卡片主题"），也将其转换为 `am config set`。设置项：`open`（自动打开浏览器）、`theme`、`mode`、`style`、`voice`（视频旁白）、`update_check`（新版本通知）。运行 `am config` 查看所有说明。

当参数以 `clean` 开头，或用户要求清理页面/缓存时：先运行 `am clean --dry-run`，并告知用户将删除多少项、多少空间。仅在用户同意后运行 `am clean`（添加 `--all` 删除所有页面和视频，`--days N` 更改保留天数）。

当参数以 `update` 开头，或用户要求更新此 skill 时：根据其安装方式进行更新。使用 `npx skills` 安装的：运行 `npx skills update answer-me-with-html -y`。作为 Claude Code 插件安装的：运行 `claude plugin update answer-me-with-html@answer-me-with-html`（或让用户在 `/plugin` → Installed 中点击 Update now），然后让用户运行 `/reload-plugins`。使用 git clone 安装的：在仓库目录中运行 `git pull && npm install`。