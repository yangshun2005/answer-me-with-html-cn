# 贡献指南

安装和测试命令位于 README 的 [开发](README.md#development) 部分。

## 基本规则

- **`skills/answer-me-with-html/scripts/am.mjs` 是生成文件。** 请编辑 `src/`，然后运行 `npm run build` 并同时提交两者。当构建产物过期时 CI 会失败。在 `am.mjs` 中遇到合并或变基冲突时，选择任意一方并重新构建；切勿手动合并它。
- **页面格式位于 `src/page.js`。** 它写入根 `<html>` 设置、旁白 `<audio>` 和末尾的 `#am-source`，而 `readPage()` 会读取它们以供 `am patch` 使用。新增的页面设置需要同时写入这两处。
- **一个主题是 `src/themes/` 中的一个文件**（`blueprint.js` 展示了格式），并在 `src/themes/registry.js` 中列出。其他所有内容都从注册表读取主题，而 `base.css` / `video.css` 从不指定主题名称。
- **一种语言是 `src/languages/` 中的一个文件**，并在 `src/languages/registry.js` 中列出。`src/language.js` 是唯一决定草稿语言的地方；页面和视频读取其结果，从不自行决定。没有对应文件的语言仍然可以渲染：`<html lang>` 是草稿声明的标签，标签文本为英文。要添加一种语言，请复制 `src/languages/en.js`，翻译标签，在注册表中列出该文件，并将该语言添加到每个内置主题的 `label` 中；`test/languages-registry.test.js` 会指出缺失的内容。需要自己字体的语言需添加 `langs` 和 `fonts`（参见 `src/languages/zh-Hant.js`）。方向来自语言标签，而非文件：从右到左的页面会获得 `dir="rtl"` 和 `src/themes/rtl.css`；`base.css` 使用逻辑属性（`inset-inline-start`、`margin-inline-end`、`text-align: start`）以同时服务于两个方向，图表则从组件上下文（`dir`）获取页面方向。`src/bidi.js` 在从右到左的文本中保持从左到右的片段（路径、`term:` 前缀、域名、带符号数字）的顺序：`render.js` 将其应用于页面正文，图表文本则通过 `svgLine` 处理。
- **重构必须保持 HTML 字节级一致。** `npm run snapshot [ref]` 使用固定时钟渲染 336 个 render / video / patch 组合，并与 `ref`（默认为 `origin/main`）进行比较。任何差异都必须是有意为之，并在 PR 中说明。
- **仓库使用英语；产品使用读者的语言。** 技能和命令说明、CLI 输出、注释和测试名称均为英语。只有面向查看者的页面和视频 UI 会被本地化（zh / en / ja / he）；中文保留在其作为主体的地方（写作检查词表、中文测试夹具和示例、README.zh-CN.md）。`test/language.test.js` 强制执行此规则。
- 仅当某行 JavaScript 中的中文是代码接受的输入，或是已有开放 Issue 的已知缺口（在原因中指明该 Issue）时，才以 `// lang-ok: <reason>` 结束该行；在 `test/language.test.js` 的 `EXCEPTIONS` 中豁免较大的合法代码块。
- **插件安装会复制整个仓库。** 请将大型或个人文件（视频、GIF、`docs/social/`）排除在 git 之外。

## Pull Request

- **先开一个 Issue，并等待维护者接受。** 描述问题和你计划的改动；对于新语法，请展示该语法。当维护者在评论中同意或将其标记为 `ready-for-human` 时，Issue 即被接受。然后开启 PR 并关联该 Issue（`Fixes #123`）。只有拼写错误和失效链接的修复可以跳过 Issue。我们会关闭没有已接受 Issue 的 PR，一旦 Issue 被接受，你可以重新打开它。
- 每个 PR 只处理一个主题。提交信息遵循 Conventional Commits（`fix:`、`feat:`、`refactor:`、`test:`、`docs:`、`chore:`）。
- 用英语撰写 PR 描述：改了什么、为什么改，以及你如何验证。
- `npm test` 必须通过，且构建产物必须重新构建。

### 审查贡献者的 PR（维护者）

1. 检查 PR 是否关联了一个已接受的 Issue。如果没有，请关闭它并附上指向上文"Pull Request"的链接。
2. 在单独的工作树中检出 PR 的 head：`git fetch origin pull/<n>/head:pr-<n> && git worktree add /tmp/pr-<n> pr-<n>`。
3. 对照 PR 的 base 审查差异（`git diff origin/main...pr-<n>`），切勿对照你当前的分支。
4. 在工作树中：运行 `npm ci`、`npm run build`（构建产物不得变化）、`npm test`，当 `src/` 有改动时运行 `npm run snapshot`。
5. 要在贡献者的分支上完成 PR，请推送到他们的 Fork。如果 PR 涉及 `.github/workflows/` 且你的令牌缺少 `workflow` 权限范围，GitHub 会拒绝推送；此时请从本仓库的一个分支开启一个中转 PR。
6. 用英语评论，然后移除工作树。

## 刷新演示视频

在渲染后的 [examples/tcp.en.md](examples/tcp.en.md)（`tcp.html`）旁边提供 [docs/demo/demo.html](docs/demo/demo.html)，以 1920×1080 打开它，等待 `window.ready`，然后调用 `window.render(i / 30)` 并截图 `frame-0000.jpg` … `frame-0719.jpg`。运行 `node docs/demo/make-demo.mjs <frames-dir>` 来添加音乐并编码。MP4 和 GIF 不提交到仓库，因为插件安装会复制整个仓库。将 MP4 上传到 GitHub 评论中，并在 README 中使用该链接。动画是确定性的，来自 [docs/demo/music.mjs](docs/demo/music.mjs) 的音乐以 120 BPM 合成，因此每次场景切换都落在节拍上。

## 发布

一次发布就是合并到 `main` 的版本号提升。所有安装都读取 `main`：`npx skills update` 获取最新提交，而 `claude plugin update` 仅在版本变化时更新。

1. `npm run release -- <x.y.z>` 将版本写入全部四个清单文件并重新构建构建产物。`test/install.test.js` 检查它们是否一致。
2. 运行 `npm test` 和 `npm run smoke:install`，然后开启并合并一个 `chore: release <x.y.z>` PR。
3. 为合并提交打标签并发布 GitHub Release：
   `git tag -a v<x.y.z> <sha> -m v<x.y.z> && git push origin v<x.y.z> && gh release create v<x.y.z> --verify-tag --notes-file <notes>`。