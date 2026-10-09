# 使用 ElevenLabs 进行旁白配音

[简体中文](elevenlabs.zh-CN.md)

`am video` 可以使用 [ElevenLabs](https://elevenlabs.io) 朗读旁白。没有密钥时它会使用系统语音，因此这项配置是可选的。

## 设置

1. 在 ElevenLabs 控制台中创建一个 API 密钥。一个仅限 **Text to Speech** 的密钥就足够了；`am` 不会调用其他任何接口。
2. 导出该密钥，然后进行渲染。

   ```bash
   export ELEVENLABS_API_KEY="sk_..."     # 将这一行添加到 ~/.zshrc 以持久保存
   am video draft.md                      # 设置了密钥时使用 ElevenLabs
   am video draft.md --voice elevenlabs   # 密钥缺失时会报错停止
   ```

Agent 只有在启动之前已设置密钥的情况下才能看到该密钥。若要始终使用 ElevenLabs 而不让 `am` 自行选择，请运行 `am config set voice elevenlabs`。

## 选择语音和模型（可选）

语音和模型适合不同的语言和偏好，因此请自行选择：

| 变量 | 设置内容 | 未设置时 |
| :--- | :--- | :--- |
| `ELEVENLABS_VOICE_ID` | 语音。从你的 ElevenLabs 语音页面复制 ID | `bIHbv24MWmeRgasZH58o`（Will，一个预置语音） |
| `ELEVENLABS_MODEL_ID` | 模型，例如 `eleven_v4_turbo`、`eleven_flash_v2_5` 或 `eleven_multilingual_v2` | `eleven_v4_turbo` |

默认值在免费计划下可处理英文和中文文本。Will 是一个英文语音；ElevenLabs 仅在 `eleven_multilingual_v2` 下将中文列为其已验证语言。若要让中文听起来像母语，请自行选择一个中文语音。有关每个模型支持的内容，请参阅[模型列表](https://elevenlabs.io/docs/overview/models)。

来自公共语音库的语音需要付费计划。在免费计划下，API 会返回 `402 paid_plan_required`。

## 费用与缓存

ElevenLabs 按字符计费。每行旁白只生成一次，并按语音、模型和文本缓存在 `~/.answer-me-with-html/cache/tts/` 中。你只需为你修改过的行，或在更改语音或模型时再次付费。

## 错误

以下是可能遇到的提示信息：

| 提示信息 | 原因 |
| :--- | :--- |
| `ElevenLabs returned 401` 或 `missing_permissions` | 密钥错误，或缺少 Text to Speech 权限 |
| `ElevenLabs returned 402` 且带有 `paid_plan_required` | 该语音是语音库语音，而你的计划是免费的；请使用预置语音或升级 |
| `Cannot connect to ElevenLabs` | 60 秒内无响应，或没有网络；添加 `--voice off` 仅生成字幕 |