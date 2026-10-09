---
description: 清理旧的 Answer me with HTML 页面、视频和旁白缓存
argument-hint: "[--days <days>] | [--all]"
allowed-tools: Bash(node *)
---

## 当前使用情况（试运行，未删除任何内容）

!`node "${CLAUDE_PLUGIN_ROOT}/skills/answer-me-with-html/scripts/am.mjs" clean --dry-run`

## 要做什么

CLI：`node "${CLAUDE_PLUGIN_ROOT}/skills/answer-me-with-html/scripts/am.mjs" clean …`

用户参数：`$ARGUMENTS`（仅接受 `--days <非负整数>` 或 `--all`；忽略其他任何内容）

- 以上是使用默认值（30 天）的试运行。当用户给出 `--days N` 或 `--all` 时，先用这些参数再次运行 `clean --dry-run`。然后使用 AskUserQuestion 询问用户是否继续，选项如下：按试运行的方式继续（推荐）/ 清理所有内容（`--all`）/ 暂时不清理。
- 用户同意后，用相同的参数（或 `--all`）再运行一次，且不带 `--dry-run`。
- 默认情况下，它会删除超过 30 天的页面和视频，并清空旁白缓存（下一个视频会重新合成旁白）。配置文件始终保留。
- 完成后，用一句话说明释放了多少空间。不要生成讲解页面。