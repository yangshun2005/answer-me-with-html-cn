# 对比

[简体中文](compare.zh-CN.md)

其他工具也能将智能体输出转化为可视化内容。最常被提及的有两个：[Archify](https://github.com/tt-a1i/archify) 和 DeepSeek Harness 的 GenUI 插件，例如 [dsh-genui](https://github.com/omdsh-dev/dsh-genui)。它们各自承担不同的职责。

| | Answer me with HTML | Archify | dsh-genui |
| :--- | :--- | :--- | :--- |
| 模型编写的内容 | Markdown 草稿 | 带类型的 JSON 规范 | `dsh-ui` 代码块中的 JSON |
| 你得到的内容 | 一个回答问题的页面 | 一张交互式图表 | 聊天回复中的组件 |
| 可承载的内容 | 文本、表格、来自你文件的真实代码、12 种组件（流程图、ER 图、时序图、树形图、时间线……） | 五种图表类型：架构图、工作流图、时序图、数据流图、生命周期图 | 30+ 种组件：卡片、表格、图表、表单、测验、3D 场景 |
| 适用环境 | 任何能运行 shell 命令的智能体：Claude Code、Codex、Cursor、OpenCode、Pi | Claude Code、Codex、opencode、Cursor；通过上传在 Claude.ai 中使用 | DeepSeek Harness Web UI |
| 结果 | 单个 `.html` 文件，无 CDN 或 Web 字体，可离线打开 | 单个 `.html` 文件 | 在聊天中展示；完成的块可导出为 HTML |
| 交互 | 在页面上评论和决策；通过一条 Reply 消息粘贴回去 | 聚焦、路径追踪、搜索、演示模式、PNG / SVG / WebM 导出 | 按钮、表单和测验将事件发送回模型；在模型编写时实时渲染 |
| 检查 | 草稿错误会附带行号和修复建议返回；ASD-STE100 写作检查 | 交付前的 Schema、布局和路径检查 | 规范守卫会丢弃错误的节点 |
| 视频 | `am video`：带旁白，3Blue1Brown 风格 | 动画图表导出 | 无 |

## 按任务选择

- **Archify**：当你想要一张系统图表，用于探索、演示或导出为图片或视频时。它的图表工具比我们的更强大。
- **dsh-genui**：当你在 DeepSeek Harness Web UI 中工作，并希望在聊天中拥有一个实时工具时：表单、测验、点击后反馈给模型。
- **Answer me with HTML**：当你在终端智能体中工作，并希望将整个答案作为一个页面呈现时：解释、对比、代码和图表。你可以阅读、保存并发送给他人。

## 这里的独特之处

- **一个页面，而非一张图表。** 一个难题需要文本和表格与图表并列呈现。智能体根据答案的形态来选择组件。
- **模型只编写内容。** CLI 负责布局、颜色和每一个坐标。在我们的[基准测试](../bench/README.md)中，一个页面草稿约 600 个 token，而针对同一问题手写 HTML 页面约需 4,900 个 token。我们未对 Archify 或 dsh-genui 进行测量。
- **无需宿主环境。** 智能体运行一条 shell 命令即可。无需在聊天应用内渲染任何内容。
- **可信赖的文本。** 代码块引用你的真实文件，写作检查确保句子简短明了。

关于 Archify 和 dsh-genui 的信息来自它们的 README，检查日期为 2026-10-07。如果此处内容已过时，请[提交 Issue](https://github.com/QingYunA/answer-me-with-html/issues)。