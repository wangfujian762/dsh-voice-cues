# dsh-voice-cues

> 带「星期五」语音包的版本是另一个仓库：`dsh-voice-cues-friday`。

回合结束、请求授权、智能体提问时，各放一段短语音提示。由宿主进程播放——页面关掉、窗口最小化、切到别的 App 都照常响。

macOS / Windows / Linux 一个包搞定，不需要装额外播放器。

只有**主会话**的回合结束才会播"任务完成"；子代理会话（`origin: subagent` 或 `delegationDepth > 0`）整类静音——完成、授权、提问都不出声。需要时可用 `includeSubagents: true` 打开。

| 时机 | DSH 事件 | 提示音 |
|---|---|---|
| 一轮对话结束 | `turn/end`（默认只认 `completed`） | `done` |
| 智能体请求授权 | `approval/asked` | `approval` |
| 智能体向你提问 | `tool/call`，工具名 `ask_user_question` | `question` |

## 安装

从插件市场装，或者：

```sh
dsh plugin --profile <你的 profile> add dsh-voice-cues
```

本地目录也行（绝对路径，装成链接）：

```sh
dsh plugin --profile <你的 profile> add /path/to/dsh-voice-cues
```

装完重启一次 DSH（桌面版重启应用），在「插件」里能看到 **语音提示音**。

## 平台

| 平台 | 播放器 | 文件 |
|---|---|---|
| macOS | `afplay`（系统自带） | `audio/*.mp3` |
| Windows | PowerShell + `System.Media.SoundPlayer`（系统自带） | `audio/*.wav` |
| Linux | `paplay` → `ffplay` → `aplay`（依次回退） | mp3 / wav |

Windows 走 `-EncodedCommand`（UTF-16LE base64）调用，路径里有空格、引号、中文都不会出问题。PowerShell 启动有 0.3–0.8 秒延迟，属正常。

## 配置

默认三档全开，默认语音是 `personal`（小鲸鱼版：小鲸鱼本轮工作完成啦 / 请给小鲸鱼点点授权哦 / 告诉小鲸鱼你的选择吧）；包内另带 `default`（通用台词）。想切换、关掉某一档或换音频，在你自己 profile 的 `cordis.patch.yml` 里加一条同 id 覆盖（整体替换，只写要改的键）：

```yaml
- id: voice-cues
  config:
    onQuestion: false
    variant: 'default'
    volume: 0.8
    turnEndReasons: ['completed', 'error']
```

| 键 | 默认 | 说明 |
|---|---|---|
| `enabled` | `true` | 总开关 |
| `onTurnEnd` | `true` | 回合结束提示音 |
| `onApproval` | `true` | 请求授权提示音 |
| `onQuestion` | `true` | 提问提示音 |
| `turnEndReasons` | `['completed']` | 哪些 `turn/end` 原因会响，可加 `error` / `aborted` / `max-tokens` |
| `includeSubagents` | `false` | 子代理会话是否也响 |
| `volume` | `1` | 0–4，1 为原音量（Windows/Linux 映射到百分比） |
| `minIntervalMs` | `600` | 同类提示音最小间隔 |
| `variant` | `'personal'` | 随包语音：`personal` 小鲸鱼（默认）、`default` 通用 |
| `audio.done` / `audio.approval` / `audio.question` | `''` | 换成自己的音频，填绝对路径 |

## 换台词 / 换音色

两套音频随包分发，直接覆盖对应目录下的同名文件即可（本地链接安装时立刻生效）。

- 小鲸鱼版（默认）：`audio/personal/done.mp3` 等；通用版：`audio/done.mp3` 等
- macOS / Linux 读 `.mp3`，Windows 读 `.wav`，两处都要换
- 也可以只改 `audio.*` 配置指向自己的文件（Windows 请给 WAV）

素材由豆包语音合成 2.0 生成（默认音色 `zh_female_vv_uranus_bigtts`，24 kHz / 64 kbps 单声道）；换成 Edge TTS 也可以，音色 `zh-CN-XiaoxiaoNeural`。离线重做：

```sh
say -v Tingting "小鲸鱼本轮工作完成啦" -o /tmp/done.aiff
ffmpeg -y -i /tmp/done.aiff -codec:a libmp3lame -b:a 48k audio/done.mp3
ffmpeg -y -i audio/done.mp3 -ar 24000 -ac 1 -c:a pcm_s16le audio/done.wav
```

## 开发

```sh
npm test        # 24 个单元测试，不发声，macOS/Windows/Linux 通用
npm run check   # 语法 + 测试
```

测试通过注入假的 spawn 覆盖三个平台的命令构造与事件路由；CI 在 macOS、Windows、Linux 上跑同一套。

## 隐私

不联网、不合成、不上报。只读会话事件里的类型与原因，音频是包内文件。

## 许可

MIT
