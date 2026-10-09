---
title: 这个仓库如何将草稿变成页面
subtitle: answer-me-with-html，渲染路径
cols: 3
---
## A 文件夹 {meta="trimmed"}
```tree list
answer-me-with-html
  bin/am.js | CLI 入口
  src
    cli.js | 命令和标志
    parse.js | frontmatter 和面板
    *render.js | 流水线
    components/ | flow、sequence、tree ...
    lint/ste.js | 写作检查
    themes/ | blueprint、shadcn、paper
    templates/ | sheet 和 doc
    runtime/ | 页面脚本
  skills/answer-me-with-html
    SKILL.md | 智能体读取的内容
    scripts/am.mjs | 打包的 CLI
```

## B 调用图 {span=2}
```flow LR
(am render) -> cli.js
cli.js -> *render.js
render.js -> parse.js: 草稿
render.js -> lint/ste.js: 行文
render.js -> components: 围栏
components -> dagre: 流程图布局
render.js -> templates & themes
templates -> [(page.html)]
group 打包进 am.mjs: cli.js, render.js, parse.js, lint/ste.js, components, templates, themes, dagre
```

## C 从哪里开始阅读 {span=3}
1. 打开 `src/render.js`。一个函数运行整条流水线。
2. 接下来阅读 `src/parse.js`。它将草稿拆分为面板和围栏。
3. 在 `src/components/` 中挑选一个组件。每个组件将文本转换为 SVG 或 HTML。