---
description: 查看或更改 Answer me with HTML 设置 — 自动打开浏览器、默认主题、浅色/深色、STE 严格程度、视频语音、更新通知
argument-hint: "[open|theme|mode|style|voice|update_check <value>] | reset [key]"
allowed-tools: Bash(node *)
---

## 当前设置

!`node "${CLAUDE_PLUGIN_ROOT}/skills/answer-me-with-html/scripts/am.mjs" config`

## 要做什么

用户参数：`$ARGUMENTS`

CLI：`node "${CLAUDE_PLUGIN_ROOT}/skills/answer-me-with-html/scripts/am.mjs" config …`

- **无参数**：使用 AskUserQuestion 让用户选择。一次最多询问 4 项设置，优先询问这些：自动打开浏览器（open）、默认主题（theme）、浅色/深色（mode）、写作检查（style）。在选项中标记当前值。用户选择后，对每项设置运行 `config set`。
- **`<key> <value>`**（例如 `open off`）：直接运行 `config set <key> <value>`。
- **`reset` 或 `reset <key>`**：运行 `config reset [key]`。
- **自然语言**（例如“停止打开浏览器”）：将其转换为匹配的键和值，然后运行。

更改后，用一两句话说明更改了什么。设置立即生效。不要生成说明页面。