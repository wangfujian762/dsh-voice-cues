# 语音提示音（dsh-voice-cues）

把"该你了"变成声音：一轮对话结束、智能体请求授权、智能体向你提问，各放一段短语音提示。由**宿主进程**播放，页面关掉、窗口最小化、切到别的 App 都照常响。

## 三个时机

| 时机 | DSH 事件 | 音频 |
|---|---|---|
| 一轮对话结束 | `turn/end`（默认只认 `completed`） | `audio/done` |
| 智能体请求授权 | `approval/asked` | `audio/permission` |
| 智能体向你提问 | `tool/call`，工具名 `ask_user_question` | `audio/question` |

子代理会话默认不响（它们结束得太频繁）；同类提示音有 600 ms 防连响。

## 平台

| 平台 | 播放器 | 文件 |
|---|---|---|
| macOS | `afplay`（系统自带） | MP3 |
| Windows | PowerShell + `System.Media.SoundPlayer`（系统自带） | WAV |
| Linux | `paplay` → `ffplay` → `aplay`（依次回退） | MP3 / WAV |

Windows 通过 `-EncodedCommand`（UTF-16LE base64）调用，路径含空格、引号或中文都不需要额外转义。PowerShell 启动带来 0.3–0.8 秒延迟，属正常。

## 安装

```sh
dsh plugin --profile <你的 profile> add dsh-voice-cues
```

本地目录安装用绝对路径。装完重启 DSH 桌面端。

## 配置

在 profile 的 `cordis.patch.yml` 写同 id 覆盖（整体替换，只写要改的键）：

```yaml
- id: voice-cues
  config:
    onQuestion: false
    volume: 0.8
```

可用键：`enabled`、`onTurnEnd`、`onApproval`、`onQuestion`、`turnEndReasons`、`includeSubagents`、`volume`（0–4）、`minIntervalMs`、`audio.done` / `audio.approval` / `audio.question`（绝对路径）。

## 换台词 / 换音色

覆盖 `audio/` 下的同名文件即可；macOS/Linux 读 `.mp3`，Windows 读 `.wav`，两者都要换。原素材是 Edge TTS 的 `zh-CN-XiaoxiaoNeural`（24 kHz / 48 kbps 单声道）。

离线重做一条：

```sh
say -v Tingting "小鲸鱼任务完成" -o /tmp/done.aiff
ffmpeg -y -i /tmp/done.aiff -codec:a libmp3lame -b:a 48k audio/done.mp3
ffmpeg -y -i audio/done.mp3 -ar 24000 -ac 1 -c:a pcm_s16le audio/done.wav
```

## 边界

- 不联网、不合成、不上报；只读会话事件的类型与原因，音频是包内文件。
- macOS 路径已在本机实测；Windows 与 Linux 的播放器调用由单元测试与 CI（macOS / Windows / Linux 三平台）覆盖，未经真机人工试听。
- 系统静音或音量归零时自然也听不到——插件不修改系统音量。
