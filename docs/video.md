# 讲解视频详情

[简体中文](video.zh-CN.md)

`am video` 所做的一切。README 只列出了最常见的要点。

`am video` 的功能：

- **逐步构建。** 当第 N 行旁白播放时，图表的第 N 步就会出现。箭头会自行绘制。如果行数多于步骤数，开头多出的行会作为引言。
- **语音旁白。** 智能体像人一样大声讲解来撰写旁白，而不是像手册那样。
- **镜头聚焦。** 旁白中的 `[Server]` 会将镜头推向该节点并高亮它。图表始终不会离开画面。
- **对象延续。** 下一场景中同名的节点会滑到它的新位置，而不是直接切换。
- **旁白。** 如果设置了 `ELEVENLABS_API_KEY`，则使用 ElevenLabs（可选；[设置指南](elevenlabs.md)），否则使用系统语音，如果两者都不存在则仅使用字幕。系统语音会根据机器上安装的语音为每一行选择：macOS 的 `say` 使用该语言已安装的语音朗读一行，繁体中文在安装了台湾语音（`zh_TW`）时会使用它；Linux 使用该语言对应的 espeak-ng 语音朗读。如果某行的语言没有安装语音，则保留其字幕而不朗读，运行时会说明是哪种语言：不会用另一种语言的语音来朗读任何内容。每个节拍持续的时间与其音频一样长，因此画面和语音保持同步。
- **本地语音。** `--voice local` 使用你自己运行的、兼容 OpenAI 的 `POST /v1/audio/speech` 服务器，该服务器返回 16 位 PCM WAV，例如搭配 Qwen3-TTS 的 [mlx-audio](https://github.com/Blaizzy/mlx-audio)。将 `AM_TTS_URL` 设置为服务器根地址；除非服务器有默认值，否则设置 `AM_TTS_MODEL` 和 `AM_TTS_VOICE`，如果服务器需要密钥则设置 `AM_TTS_API_KEY`。`AM_TTS_EXTRA` 是一个 JSON 对象，包含合并到每个请求中的模型特定选项；`AM_TTS_MODEL` 和 `AM_TTS_VOICE` 会覆盖其中的同名字段，而 `am` 始终会设置 `input`、`response_format` 和 `stream`。如果某个片段比其文本所暗示的时长明显更短或更长，则会重新生成，最多尝试 `AM_TTS_ATTEMPTS` 次（默认 3；设为 1 则关闭检查）。`am patch` 会保留视频制作时所用的语音。示例：`AM_TTS_URL=http://127.0.0.1:8000 AM_TTS_MODEL=mlx-community/Qwen3-TTS-12Hz-1.7B-CustomVoice-4bit AM_TTS_VOICE=vivian am video draft.md --voice local`。
- **与页面外观一致。** 视频默认使用蓝图绘制风格：带横线的边框和字母编号的图纸标题。写入 `theme: 3b1b` 可获得深色 3Blue1Brown 外观。`theme: shadcn` 和 `mode: dark` 也可用。
- **章节和速度。** 播放器控件上方有一条章节条：每个场景一个标签，当前章节高亮显示，点击可跳转到该场景并继续播放。进度条上的刻度也可点击跳转。速度按钮在 0.5× 到 2× 之间循环；旁白会随画面一起拉伸。
- **单个文件。** 页面内嵌音频，可离线播放。播放器的导出按钮（⤓）会在浏览器中编码同一个视频并下载它——无需终端，无需 ffmpeg——并显示进度，点击封面可停止。它需要安全上下文（本地文件或 `localhost`）：通过纯 HTTP 提供的页面无法使用 WebCodecs。添加 `--mp4` 可在页面旁生成 1080p MP4（H.264 + AAC）。它需要 ffmpeg，耗时约为视频时长的 1.3 倍。添加 `--webm` 可生成由页面自行编码的同一视频的 WebM（VP9 + Opus）。它不需要 ffmpeg；其耗时取决于页面有多少内容在动（实测：11 秒的草稿耗时 28 秒，一个 2:27、含五个场景的讲解视频在笔记本 CPU 上以 1080p30 耗时 13.5 分钟）。两者都需要 Chrome 和 Node.js 22+，并且可以同时给出这两个标志。

示例视频的草稿（[examples/video-tcp.en.md](../examples/video-tcp.en.md)，约 45 秒）为 1.3 KB，几百个输出 token。渲染在无语音时不到一秒，使用系统语音时约 4 秒。智能体只在你要求时才制作视频；常开模式仍然只制作页面。完整语法：`am help video`。