# 参考

[简体中文](reference.zh-CN.md)

## 草稿格式

````markdown
---
template: sheet        # sheet = 面板网格（默认），doc = 带目录的单栏
theme: auto            # auto（默认）为长文本选择 paper，为图表选择 blueprint；或 blueprint、shadcn、paper、你自己的主题
title: 页面标题
subtitle: 一行副标题
cols: 3                # sheet 的列数
source: RFC 9293       # 任何其他字段显示在标题下方
---
一两句话说明要点。

## 面板标题 {span=2 meta="小字，右上角"}
普通 Markdown：段落、列表、表格。

```flow LR
A -> B: 标签
```
````

- 每个 `## ` 标题都是一个面板。字母 A、B、C 是可选的，会自动为你添加。
- `span=2` 使面板宽两列，`rows=2` 使其高两行，`bare` 移除其标题栏。`span` 只是一个提示：页面会根据内容调整每个面板的大小，因此宽表格和图表无需 `span`。只有需要突出的面板才写 `span`。`rows` 仅适用于普通网格（无 JavaScript 时、打印时和窄屏幕上）；浏览器中的两端对齐布局会忽略它。
- 当没有组件适用时，使用 ```` ```html ```` 或 ```` ```svg ```` 块嵌入原始标记。
- 诸如 `<host>` 之类的占位符会显示为文本。在句子内部，只有文本级标签会保留（`b`、`kbd`、`sup`、`a`、`span`、`br`、`img`……），而破坏页面的标签（`script`、`style`、`iframe`……）也会显示为文本。将原始标记放在 `html` 或 `svg` 块中，将代码放在反引号中。渲染结果会列出它更改的每个标签。

组件的完整语法：`am help <component>`。

## 在页面上回答

每个页面都有一个 **Reply** 按钮。读者可以对任何面板发表评论（其标题栏中的对话气泡按钮），并将一条 Markdown 回复复制回给 agent。回答和评论在重新加载后仍然保留。

对于读者必须做出的决定，在其影响的面板中写一个 `ask` 块：

````markdown
```ask
我们使用哪个缓存？
* Redis | 重启后保留数据
- Memcached | 更简单，无磁盘
```
````

- 第一行是问题。每个选项以 `*`（建议项，开始时被选中）或 `-` 开头。写 2 到 6 个选项。
- 单选需要恰好一个 `*`。`ask multi` 允许读者选择多个；`*` 标记开始时被选中的选项。
- 回复会说明每个决定读者是更改了它、确认了建议，还是没有回答。评论文本会以 `>` 引用返回。

完整语法：`am help ask`。

## 代码块

引用项目中已有的代码，而不是手动输入：

````markdown
```ts src=server/routes.ts lines=18-30 hl=22
```
````

- 任何语言不是组件的围栏都是代码块，带有标题栏和复制按钮。
- `src=` 读取文件，`lines=18-30` 选取行，按文件中的行号编号。将块留空。路径从当前文件夹读取，只有其中的文件会被引用。
- `hl=22` 或 `hl=20-22,25` 高亮行。对于你输入的代码，`title="limits.ts · sketch"` 为其命名，`start=38` 从第 38 行开始编号。
- 超过 40 行的块会收到警告；超过 200 行是错误。按约定存放密钥的文件（`.env`、`*.pem`、`id_rsa`、`.ssh/`、`.git/`……）以及任何看起来像密钥或令牌的文件都会被拒绝。规则遵循 html-plan 的规则。
- 渲染结果会列出它嵌入的每个文件。页面保留路径；`am patch` 会再次读取文件，或在文件已移动时保留页面的副本。

要显示更改，将统一 diff 粘贴到 `diff` 围栏中，CLI 会绘制它：

````markdown
```diff file=src/code.js
@@ -60,3 +60,3 @@
 export function parseCodeArgs(args) {
-  const attrs = parseAttrs(args);
+  const attrs = parseAttrs(args, { diff: true });
   const opts = {};
```
````

- 以 `+` 开头的行是新增（绿色），`-` 是删除（红色），空格是上下文。`@@ -60,3 +60,3 @@` 开始一个块：它显示为细行并设置行号，在两个行号槽中（旧和新）。没有 `@@` 时，`start=N` 从 N 开始编号；两者都没有时，块没有行号。
- `file=` 在标题栏中命名文件，语言标签来自其扩展名。当缺少 `file=` 时，`+++ b/path` 行会命名它。标题栏还会显示诸如 `+1 −1` 的统计信息。复制会按原样复制 diff。
- `hl=` 使用新侧行号。`diff --git`、`---`、`+++` 和 `\ No newline at end of file` 行会被接受但不绘制。
- 不以 `+`、`-`、空格或 `@@` 开头的行是错误，截断（`...`）也是：将 diff 拆分为两个块。计数与 `@@` 头不匹配的块是警告。`src=` 不支持与 `diff` 一起使用：请粘贴 diff。限制与其他代码块相同。

完整语法：`am help code`。

## 图片

对于图表无法展示的内容，例如真实屏幕，使用已作为文件存在的图片：

```markdown
![游戏的三种查看模式](/absolute/path/to/screenshot.png)
```

- 将图片单独放在其行上。alt 文本成为其说明文字，因此写出图片展示的内容。写作检查会读取它。
- 使用绝对路径。相对路径从草稿文件所在文件夹读取，或当草稿来自 stdin 时从当前文件夹读取。路径中的空格没问题。
- PNG、JPG、GIF、WebP、AVIF 和 SVG 文件，最大 5 MB。文件被嵌入页面，因此页面保持为可离线打开的单个文件。宽图片会缩小到其面板。
- `http(s)` URL 和 `data:` URI 保持原样。URL 在打开页面时需要网络。
- 页面保留每张图片的路径。`am patch` 会从文件再次嵌入图片，或在文件已移动时从页面嵌入。
- 该工具不生成图片。

完整语法：`am help image`。

## 直接使用 CLI，无需 agent

CLI 是技能文件夹中的 `scripts/am.mjs`。

````bash
AM=skills/answer-me-with-html/scripts/am.mjs

node $AM render examples/tcp.en.md                # 渲染并在浏览器中打开
node $AM render notes.md -o out.html --no-open    # 选择输出文件，不打开
node $AM render notes.md --theme shadcn           # 为本次运行选择主题
node $AM patch page.html --panel "Why three messages" < panel.md   # 替换一个 ## 面板，覆盖同一文件
node $AM lint notes.md                            # 仅写作检查
node $AM list                                     # 列出组件
node $AM config                                   # 查看设置

# 从 stdin 读取。这是 agent 调用它的方式。
node $AM render - <<'AM_EOF'
## A One panel
```flow
A -> B: hello
```
AM_EOF
````

页面默认输出到 `~/.answer-me-with-html/pages/`。设置 `AM_HOME` 可移动它们。默认文件夹对你的用户私有（模式 0700），因此共享主机上的其他用户无法读取你的页面；已存在的 `AM_HOME` 文件夹保持其模式。

## 语言

草稿的语言设置页面的 `lang` 属性、按钮和主题名称的语言、字体以及视频播放器标签。草稿可以在头部用 `lang:` 声明它。没有它时，工具从文本的书写系统检测语言。

| `lang:` | `<html lang>` | 标签和字体 |
| :--- | :--- | :--- |
| `zh`、`zh-CN`、`zh-Hans` | 按所写（单独的 `zh` 变为 `zh-CN`） | 简体中文 |
| `zh-Hant`、`zh-TW`、`zh-HK`、`zh-MO` | 按所写 | 繁体中文，优先使用繁体字体 |
| `en`、`en-US` 和其他英语标签 | 按所写 | 英语 |
| `ja`、`ja-JP` | 按所写 | 日语，优先使用日语字体 |
| `he`、`he-IL` | 按所写 | 希伯来语，优先使用希伯来语字体；页面从右到左 |
| 任何其他标签，如 `fr` 或 `ko` | 按所写 | 英语标签 |

标签按你声明的方式书写：`zh-tw` 和 `zh_TW` 都变为 `zh-TW`。空值、`und` 或无效值会被忽略，由文本决定。

从右到左的语言（`he`、`ar`、`fa`、`ur`、`yi`）还会获得 `<html dir="rtl">`。页面会镜像：文本、面板、表格、目录和工具栏从右侧开始；`flow` 和 `sequence` 图表绘制为其镜像，因此第一个节点或参与者位于右侧，箭头指向左侧；时间线、树和限制条从右侧开始。代码块、diff 和行内代码保持从左到右，任何没有从右到左字母的文本串也是如此（诸如 `src/` 的路径、标签开头诸如 `lint:` 的键、域名、带符号的数字），因此其标点保持在书写的位置。希伯来语有自己的标签、常见 frontmatter 键的希伯来语名称（`author`、`date`、`source`、`version`……）以及日.月.年的渲染日期；其他从右到左的语言使用英语标签。

| 文本中的书写系统 | 检测为 |
| :--- | :--- |
| 汉字 | `zh-CN`（简体），或当仅以繁体形式书写的字符多于仅以简体形式书写的字符时为 `zh-Hant` |
| 汉字带假名 | `ja` |
| 谚文 | `ko` |
| 西里尔文、阿拉伯文、希伯来文、泰文、希腊文 | `ru`、`ar`、`he`、`th`、`el`（该书写系统最常见的语言） |
| 拉丁文 | `en` |

当你书写英语以外的拉丁字母语言、当中文文本太短或太简单而无法显示繁体字符（两种形式共有的字符说明不了什么，文本保持简体），或当确切语言很重要时（阿拉伯文也书写波斯语，西里尔文也书写乌克兰语），请声明语言。

已修补的页面保持其原有语言，除非草稿声明了语言。

图表文本按书写系统使用的单位换行：一个汉字、假名或全角字符，书写系统使用空格时为一个单词，对于不使用空格的书写系统（泰文、老挝文、高棉文和缅甸文）为一个词典单词，因此长单词不会将节点推得超出其预算。单独无法放入行的单词回退到字素，因此组合标记与其基础字符保持在一起。

写作检查遵循语言。中文、英语和日语保持其规则。任何其他语言只获得语言中立的规则：句子长度（以单词计，或 CJK 文本以字符计）和段落长度。不在单词之间书写空格的书写系统按分词器找到的单词计量，与图表换行使用的四种书写系统相同。

## 你自己的主题

在 `~/.answer-me-with-html/themes/` 中为每个主题放一个 JSON 文件。文件名即主题名，因此 `themes/notes.json` 是主题 `notes`。像内置主题一样选择它：草稿中的 `theme: notes`、`--theme notes` 或 `am config set theme notes`。agent 编写相同的草稿，因此主题不会为每个回答增加任何内容。

```json
{
  "label": "Notes",
  "tokens": {
    "common": { "--font-sans": "\"IBM Plex Sans\", \"Noto Sans CJK SC\"" },
    "light": { "--bg": "#f7f5ef", "--paper": "#fffdf8", "--ink": "#1f1d1a" },
    "dark": { "--bg": "#14130f", "--paper": "#1c1b17", "--ink": "#eeeae0" }
  },
  "css": "& .am-panel-head { letter-spacing: 0.01em; }"
}
```

- 浅色和深色必须各自设置每个颜色变量。`am help theme` 列出它们并展示完整格式。
- 只命名已安装的字体。默认字体会作为回退添加，因此没有你的字体的读者仍能获得可读的文本。
- 在 `css` 中，每个选择器都以 `&` 开头。它代表主题的根，因此规则只在此主题下适用。
- 每个页面都携带内置主题及其自己的主题，因此它仍可在任何机器上打开。
- `am theme check notes` 报告缺失的变量、无效的颜色以及浅色和深色下的低对比度，并渲染两个包含每个组件的样本页面。

## 从远程机器打开页面

如果你的 agent 在远程或无头机器上运行，`am serve` 让你通过 `http://` 在自己的浏览器中打开其页面。在该机器的终端中运行它并保持运行：

````bash
am serve              # http://127.0.0.1:8765，Ctrl-C 停止；--port N 更改端口（0 选取一个空闲端口）
ssh -L 8765:127.0.0.1:8765 user@host   # 在你自己的计算机上，然后在浏览器中打开链接
````

- 当它运行时，`am render`、`am patch` 和 `am video` 会为它们写入 `~/.answer-me-with-html/pages/` 或 `videos/` 的每个页面打印一行 `link:`，agent 会给你该链接而不是 `file://` 链接。
- 服务器仅监听 `127.0.0.1`。使用 SSH 隧道访问它；不要将端口暴露到互联网或共享网络。没有密码，也没有 TLS。
- 链接是按页面的：每个链接携带该页面的令牌，它不会打开任何其他页面。服务器重启后它们停止工作。服务器不显示文件夹列表，只提供 `.html` 文件。
- 将链接视为其页面的密码：任何能访问端口并拥有链接的人都可以打开页面，包括同一机器的其他用户。只与你本人分享。
- 给隧道一个自己的本地端口。不要重用你从另一服务器或另一主机打开页面的端口：来自那台其他服务器的页面可能已为你的浏览器中该地址留下了 service worker，它会看到之后在那里提供的页面。
- 每个页面在其自己的浏览器沙箱中打开，因此一个页面中的脚本无法读取另一个。因此，`http://` 链接上的 Reply 回答和评论只在你关闭页面之前有效：在关闭之前复制回复。

## 更新和清理

**更新。** 更新是手动的，工具会在有更新时告诉你。每周一次，后台进程从 GitHub 下载本项目的 `package.json` 以读取最新版本号。不会发送关于你或你的页面的任何内容，你请求的页面从不等待它。当有更新版本时，下一次渲染会添加一行通知，agent 会询问你是否要更新。用 `/answer-me-with-html:config update_check off` 关闭它。

| 安装方式 | 更新方式 |
| :--- | :--- |
| `npx skills add` | `npx skills update answer-me-with-html -y`，或告诉你的 agent "update answer-me-with-html" |
| `git clone` + `npm link` | 在仓库中执行 `git pull && npm install` |
| Claude Code 插件 | 在终端中运行 `claude plugin update answer-me-with-html@answer-me-with-html`（或打开 `/plugin` → Installed → Update now），然后 `/reload-plugins`。要自动更新，在 `/plugin` → Marketplaces 中为此市场开启自动更新。Claude Code 对第三方市场默认关闭它 |

**清理。** 页面、视频和旁白缓存会在 `~/.answer-me-with-html/` 中累积。如果该文件夹增长超过 200 MB，或在 30 天内未清理而超过 20 MB，agent 会每周询问一次是否清理。未经你同意不会删除任何内容。

- `/answer-me-with-html:clean`（插件），或说 "clean up the pages"，会先预览然后询问。
- `am clean` 删除超过 30 天的页面和视频并清空旁白缓存。`--days N` 更改截止时间，`--all` 删除每个页面和视频，`--dry-run` 只显示将删除的内容。你的设置和主题始终保留。如果 `pages`、`videos` 或 `cache` 本身是符号链接，则跳过：`am clean` 从不触碰它指向的文件，大小计数和清理提示也忽略它。无法读取的文件夹也会跳过：其文件不计入大小计数和清理提示，因此你看到的大小可能低于实际使用量，`am clean` 不会删除它们。

**旧的常开插件。** 它已从本仓库移除，但你的副本会继续添加提醒，直到你移除它。运行 `/plugin uninstall answer-me-with-html-always@answer-me-with-html`，然后从 [README](../README.md#always-on-mode-recommended) 添加常开规则。

## 设置

| 键 | 默认值 | 作用 |
| :--- | :--- | :--- |
| `open` | `on` | 页面生成后在浏览器中打开每个页面。如果弹窗打扰你，请关闭它 |
| `